---
layout: post
title: unix bash快捷键
date: 2019-05-17 
tags: Unix
---

## Linux/Unix下Shell快捷键操作集合

### 快速搜索
1) ctrl + r 组合键来进入历史搜索模式在history表中查询某条过往指令，找到需要重复执行的命令后，按回车键即可。
2) !!：重复执行上一条指令
3) !a：重复执行上一条以a为首的指令
4) !number：重复执行上一条在history表中记录号码为number的指令
5) !-number：重复执行前第number条指令
6) !$：表示获得上一条命令中的最后一项内容。e.g 先来看一个例子： mkdir /exampledir  && cd !$ 


### 编辑的快捷键
1) Ctrl + a：将光标定位到命令的开头
2) Ctrl + e：将光标定位到命令的结尾，与上一个快捷键相反
3) Ctrl + u：剪切光标之前的内容
4) Ctrl + k：剪切光标之后的内容，与上一个快捷键相反
5) Ctrl + y：粘贴Ctrl + u和Ctrl + k所剪切的内容
6) Ctrl + t：交换光标之前两个字符的顺序
7) Ctrl + w：删除光标左边的参数（选项）或内容
8) Ctrl + l：清屏
9) Ctrl + d：输入已结束。在shell下相当于一个exit
10) Ctrl + c：键盘中断请求。
11) Ctrl + s & Ctrl + q：暂停/恢复屏幕输出
12) Ctrl + n(↓)：显示下一条命令
13) Ctrl + p(↑)：显示上一条命令
14) Ctrl + b：向回移动
15) Ctrl + f： 向前移动
16) Ctrl + shift +↓：终端向下滚动
17) Ctrl + shift +↑：终端向上滚动
18) Shift+pgup/pgdown：终端上下翻页滚动

### 处理作业
首先，使用 Ctrl + z 快捷键可以让正在执行的命令挂起。如果要让该进程在后台执行，那么可以执行 bg 命令。而 fg 命令则可以让该进程重新回到前台来。使用 jobs 命令能够查看到哪些进程在后台执行。 你也可以在 fg 或 bg 命令中使用作业 id，如： fg %3 又如： bg %7  

### 使用管道grep
例子：
 * ps aux | grep init
 * ps aux | tee filename | grep init
 * ps aux | tee -a filename | grep init
 
### 将标准输出保存为文件
你可以将命令的标准输出内容保存到一个文件中，举例如下： ps aux > filename 注意其中的“>”符号。 你也可以将这些输出内容追加到一个已存在的文件中： ps aux >> filename 