---
title: "Linux磁盘空间监控"
date: 2020-04-17T13:02:09+08:00
draft: false
categories: ["Linux"]
tags: ["Linux", "硬盘", "statvfs"]
featured_image: 
author: "Tifinity"
---

### Linux磁盘空间监控

>开发当牛 测试做马
>测试开发 当牛做马

我要做的工作基本上都是学习这位**天外归云**大佬的：[天外归云的博客](https://www.cnblogs.com/LanTianYou/)，上面引用的也是大佬首页上的话，觉得很有意思hh。下面正式开始。

### 获取磁盘信息

首先Linux查看磁盘空间的命令有两个：

- du（disk usage 硬盘使用率）：显示每个文件和目录占用的空间，如果直接执行du，你就会看到一长串文件刷出来。

- df （disk free 空余硬盘）：显示磁盘分区上可以使用的磁盘空间。

  ![](https://raw.githubusercontent.com/Tifinity/MyImage/master/UsageMonitor/20200415113432.png)

那么df就可以满足我们的需求，现在在python中调用df：

可以用commands或者subprocess，commands模块在Python3中被subprocess取代。

~~~python
import subprocess
code = subprocess.call(['df'])
print(code)
# 输出与在命令行中使用df一样，如果执行成功的话code的值是0
~~~

使用subprocess确实能够输出与df一样的结果，实际上就相当于创建了一个子进程执行了df，不过这个结果好像获取不到，没法进行操作，因为subprocess.call()只返回一个状态码表示完成情况，需要想别的办法。

#### 第一种方法

就是用上面的df指令，并想办法获取它的结果来进行操作。

目前对Linux的指令还不熟悉，挖个坑，以后补充·~·

#### 第二种方法

其实Python中的os包已经有方法可以得到磁盘信息了。

>在所给的路径上执行 `statvfs()` 系统调用。返回值是一个对象，其属性描述了所给路径上的文件系统，并且与 `statvfs` 结构体的成员相对应，即：`f_bsize`, `f_frsize`, `f_blocks`, `f_bfree`, `f_bavail`, `f_files`, `f_ffree`, `f_favail`, `f_flag`, `f_namemax`, `f_fsid`。

该方法的使用参考[菜鸟教程](https://www.runoob.com/python/os-statvfs.html)和[中文文档](https://docs.python.org/zh-cn/3.7/library/os.html#module-os)

~~~python
def get_disk_usage_percent():
    statvfs = os.statvfs('/')
    # f_frsize: fragment size碎片大小
    # f_blocks: 文件系统数据块总数
    # f_bfree: 文件系统可用数据块数
    total_disk_space = statvfs.f_frsize * statvfs.f_blocks
    free_disk_space = statvfs.f_frsize * statvfs.f_bfree
    disk_usage = (total_disk_space - free_disk_space) * 100.0 / total_disk_space
    disk_usage = int(disk_usage)
    return disk_usage
~~~

所以只需计算一下空闲空间和总空间，即可得到磁盘使用率。

### 参考

[python语言获取linux磁盘使用情况](https://blog.csdn.net/magic_zj00/article/details/7207445)

[python监控linux磁盘空间使用情况并邮件通知管理员](https://blog.csdn.net/shangshaohui2009/article/details/79628684)

[Linux服务器CPU，内存，磁盘空间，负载情况查看python脚本](https://www.cnblogs.com/LanTianYou/p/6913833.html)

[Linux proc详解](https://www.iteye.com/blog/luckyclouds-675711)

[【Linux】/proc/stat详解 完整验证版](https://blog.csdn.net/zd199218/article/details/80698192)

