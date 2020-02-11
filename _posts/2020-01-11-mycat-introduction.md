---
layout: post
title: mycat 数据库分库分表中间件学习
date: 2020-01-11
tags: mycat
---
## 安装与启动
> 安装

官网：http://www.mycat.io/，我下载的是mac版本http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20180122220033-mac.tar.gz

mycat是基于java的，需要环境依赖，我安装了openjdk1.8的
```
➜  mycat java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

解压压缩包，tar -zxvf Mycat-server-1.6.5-release-20180122220033-mac.tar.gz

> 启动

进入文件夹内
```
cd mycat
```

启动
```
➜  mycat bin/mycat start
Starting Mycat-server...
```

查看启动是否成功
```
➜  mycat more logs/wrapper.log
STATUS | wrapper  | 2020/01/11 15:21:51 | --> Wrapper Started as Daemon
STATUS | wrapper  | 2020/01/11 15:21:51 | Launching a JVM...
INFO   | jvm 1    | 2020/01/11 15:21:51 | Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=64M; support was removed in 8.0
INFO   | jvm 1    | 2020/01/11 15:21:53 | Wrapper (Version 3.2.3) http://wrapper.tanukisoftware.org
INFO   | jvm 1    | 2020/01/11 15:21:53 |   Copyright 1999-2006 Tanuki Software, Inc.  All Rights Reserved.
INFO   | jvm 1    | 2020/01/11 15:21:53 | 
INFO   | jvm 1    | 2020/01/11 15:21:55 | MyCAT Server startup successfully. see logs in logs/mycat.log
```




## 配置说明

> server.xml

`作用：`

1、配置系统相关参数

2、配置用户访问权限
```
    <user name="root" defaultAccount="true">
            <property name="password">123456</property>
            <property name="schemas">TESTDB</property>

            <!-- 表级 DML 权限设置 -->
            <!--            
            <privileges check="false">
                    <schema name="TESTDB" dml="0110" >
                            <table name="tb01" dml="0000"></table>
                            <table name="tb02" dml="1111"></table>  //dml 四位数依次insert,update,select,delete
                    </schema>
            </privileges>           
             -->
    </user>

    <user name="user">
            <property name="password">user</property>
            <property name="schemas">TESTDB</property>
            <property name="readOnly">true</property>
    </user>
```

3、配置SQL防火墙及SQL拦截功能


> log4j2.xml 

`作用：`

1、配置输出日志的格式

2、配置输出日志的级别

> rule.xml 

`作用：`

1、配置水平分片的分片规则

2、配置分片规则所对应的分片函数

- 简单取模partitionByMod
- hash取模partitionByHashMod
- 枚举分片partitionByFileMap

> schema.xml 

定义逻辑库具体配置
```
 <table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2,dn3" />
```
primaryKey如果分片的分片规则不是定义的值，会另外建立主键索引。

```
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <!-- can have multi write hosts -->
        <writeHost host="hostM1" url="localhost:3306" user="root"
                           password="123456">
                <!-- can have multi read hosts -->
                <readHost host="hostS2" url="192.168.1.200:3306" user="root" password="xxx" />
        </writeHost>
        <writeHost host="hostS1" url="localhost:3316" user="root"
                           password="123456" />
        <!-- <writeHost host="hostM2" url="localhost:3316" user="root" password="123456"/> -->
</dataHost>
```
balance="0"

- 0 不开启读写分离机制
- 1 全部的readHost 与 stand by writeHost参与select语句的负载均衡
- 2 所有的readHost与writeHost都参与select语句的负载均衡
- 3 所有readHost参与select语句的负载均衡

writeType="0" 

- 0：默认一台写，高可用，down了之后切换到指定
- 1：随机

## 垂直分库
- 1、收集分析业务模块间的关系
- 2、复制数据库到其他实例
- 3、配置mycat垂直分库
- 4、通过mycat访问DB
- 5、删除原库中已迁移表



## 水平分库

## 迭代
* 2020年02月11日 21:54:00 初稿

## 参考