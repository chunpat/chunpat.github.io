---
layout: post
title: elasticsearch生产部署
date: 2019-09-09 
tags: elasticsearch
---

## 1、安装OpenSdk
sudo apt-get install openjdk-8-jdk

配置Java环境变量
sudo vim /etc/profile
在profile末尾添加以下内容:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

```

添加后保存并退出，用java -version查看是否配置成功
```
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-8u191-b12-2ubuntu0.18.04.1-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)

```

## 2、安装Elasticsearch 

选择相应版本，大小200多M，有点大
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.1-linux-x86_64.tar.gz
```

解压
```
tar -zxvf elasticsearch-7.3.1
```

## 3、安装elasticsearch-ik分词

```
sudo ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.3.1/elasticsearch-analysis-ik-7.3.1.zip
```


## 4、改配置 /elasticsearch/bin/elasticsearch.yml


## 参考
* 1、[Elasticsearch和Head插件安装 - v-imok - 博客园](https://cnblogs.com/xiaojianfeng/p/9435507.html)

* 2、[请教elasticsearch出现unassigned shards根本原因 - Elastic 中文社区](https://elasticsearch.cn/question/4136)

* 3、[官方index检测](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/cluster-allocation-explain.html


3、yellow问题
https://discuss.elastic.co/t/cluster-yellow-reason-shards-started-kibana-0/93034

4、kibana
https://www.elastic.co/guide/en/kibana/current/targz.html