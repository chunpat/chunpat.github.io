---
layout: post
title: PHP缓冲区
date: 2019-09-07
tags: nginx,php,缓冲区
---

## 事由
上一篇文章中，写了关于nginx和php-fpm组合 请求超时的解决方案,
设置了保持长时间请求不断。但是，又遇到个问题就是缓冲区，需要边保持请求连接边打印东西
到浏览器。

## 解决
涉及php.ini属性概念与修改
output_buffering
implicit_flush
output_handler

### cli模式的缓冲区强制不开启
output_buffering 为off
implicit_flush 为on

### php-fpm 才可以用php.ini的设置

默认设置是 output_buffering=4096，implicit_flush 为off,所以要缓冲区输出的话就要手动填充或者改配置，然后flush出来。

```
    for ($i = 0; $i< 10;$i++){
        ob_start();
        //查看执行情况
        echo '完成:' .$i . '/100' . PHP_EOL;
        echo str_repeat(" ", 1024 * 2);//人为将缓冲数据扩充到2k

        ob_end_clean();
        sleep(1);
    }
```

## 参考

1、[关于PHP 刷新缓冲区操作(边执行边输出)简单分析](https://www.cnblogs.com/devcjq/articles/6072945.html)
2、[深入理解php的输出缓冲区(output buffer) - EzrealR - 博客园](https://www.cnblogs.com/raobenjun/p/8086051.html)
3、[php缓冲区 - EzrealR - 博客园](https://www.cnblogs.com/raobenjun/p/8085957.html)

