---
layout: post
title: elasticsearch学习
date: 2019-09-02 
tags: elasticsearch
---
### 安装
下载相应版本 https://github.com/elastic/elasticsearch/releases


### 配置
修改配置，其他主机可以访问，单节点部署
```
vim config/elasticsearch.yml
```

![FhoCc45P5ISRy7oFsr5ZQLdhK-3P](http://img.zzhpeng.cn/FqYRhHL4hv5NrrJMumI_wtDGIng1)

### lk分词插件
中文分词器，同lucene一样，在使用中文全文检索前，需要集成IK分词器。找到相应的发行版本安装。
```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.3.1/elasticsearch-analysis-ik-7.3.1.zip

```
如果报错，多数是网络问题，文件没下载完整。

### Kibana图形管理安装


### 实战
官方开发的php组件

php客户端
https://github.com/elastic/elasticsearch-php

说明文档
https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/overview.html

关于查询
https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html
