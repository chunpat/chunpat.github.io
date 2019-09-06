---
layout: post
title: PHP缓冲区
date: 2019-09-07
tags: nginx,php,缓冲区
---

## 事由
上一篇文章中，写了关于nginx和php-fpm组合 请求超时的解决方案。
但是，又遇到个问题就是缓冲区，需要边保持请求连接边打印东西
到浏览器。


## 解决


## 参考

1、[关于PHP 刷新缓冲区操作(边执行边输出)简单分析](https://www.cnblogs.com/devcjq/articles/6072945.html)
2、[深入理解php的输出缓冲区(output buffer) - EzrealR - 博客园](https://www.cnblogs.com/raobenjun/p/8086051.html)
3、[php缓冲区 - EzrealR - 博客园](https://www.cnblogs.com/raobenjun/p/8085957.html)

