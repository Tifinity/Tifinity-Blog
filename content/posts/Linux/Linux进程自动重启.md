---
title: "Linux进程自动重启"
date: 2020-04-22T17:57:06+08:00
draft: true
categories: ["Linux"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# Linux通过Shell脚本控制进程自动重启

接着上一篇，解决最后一个问题：监控程序如何保活？

监控程序需要一直运行才能实现监控的目的，但是监控程序本身也是一个进程，他会因为各种原因挂掉，所以怎么保持监控程序存活就比较重要了。

一开始的想法是这样的：

写一个脚本，在一个无限循环里不断找指定进程，看看还在不在运行，如果不存活则重新启动该程序，并记录下来。然后发现这个脚本一运行不也是一个进程，那他自己不是也会挂掉，这样用一个自己都不能保活的进程来实现保活显然不对。

那么只能从操作系统的层面来实现了，于是找到了 crontab 这个命令，Linux中定期执行程序的命令，这样就可以实现了。

脚本基本上参考[linux 进程监控和自动重启的简单实现](https://www.cnblogs.com/fatt/p/4974756.html)，在做的过程遇到很多问题，







### 参考

[菜鸟教程](https://www.runoob.com/linux/linux-comm-crontab.html)

[linux 进程监控和自动重启的简单实现](https://www.cnblogs.com/fatt/p/4974756.html)