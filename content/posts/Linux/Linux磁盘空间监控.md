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

## 获取磁盘信息

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

## 通知管理员

那么，磁盘空间不足之后，就需要通知管理员了，使用[上一篇博客]([https://tifinity.github.io/2020/python%E5%A6%82%E4%BD%95%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6/](https://tifinity.github.io/2020/python如何发送邮件/))的方法发送邮件即可。

Github项目地址：[UsageMonitor](https://github.com/Tifinity/UsageMonitor)

~~~python
def get_disk_usage_percent():
    statvfs = os.statvfs('/')
    # f_frsize: 分栈大小
    # f_blocks: 文件系统数据块总数
    # f_bfree: 文件系统可用数据块数
    total_disk_space = statvfs.f_frsize * statvfs.f_blocks
    free_disk_space = statvfs.f_frsize * statvfs.f_bfree
    disk_usage = (total_disk_space - free_disk_space) * 100.0 / total_disk_space
    disk_usage = int(disk_usage)
    return disk_usage

def monitor_warning():
    sender = '1139390530@qq.com' #发送者
    receivers = ['1139390530@qq.com']  # 接收者
    subject = '坏球了'
    content = '坏球了' 
    email_send.send_mail(sender, receivers, subject, content)

if __name__ == '__main__':
    while True:
        disk_usage = get_disk_usage_percent()
        disk_tip = "硬盘空间使用率（最大100%）："+str(disk_usage)+"%"
        print(disk_tip)
        max_usage = 80
        if disk_usage > max_usage:
            monitor_warning()
        time.sleep(10)
~~~

同样的，除了磁盘空间还可以自己完成CPU，内存等信息的监控和通知。

## 总结

写这样一个程序其实是面试的时候面试官给我布置的"课后作业"，让我完成一个监控服务器磁盘空间的程序并用在项目中，上面的项目简单完成了这个功能，也放进Docker里测试过了，不过还有一个问题就是监控程序保活的问题还没解决，目前正在学习。

然后之后要是真正使用感觉还是用Monit这类监控软件比较好，毕竟自己写的还很粗糙，不过在过程中学习到很多东西，也算是稍微接触了下运维的工作，对自己有好处。

尴尬的是，面试结束后太激动手一抖就把腾讯会议关了，没记下面试官的邮箱😂，要是您看到这篇博客，说明我没偷懒哈·~·😋



## 参考

[python语言获取linux磁盘使用情况](https://blog.csdn.net/magic_zj00/article/details/7207445)

[python监控linux磁盘空间使用情况并邮件通知管理员](https://blog.csdn.net/shangshaohui2009/article/details/79628684)

[Linux服务器CPU，内存，磁盘空间，负载情况查看python脚本](https://www.cnblogs.com/LanTianYou/p/6913833.html)

[Linux proc详解](https://www.iteye.com/blog/luckyclouds-675711)

[【Linux】/proc/stat详解 完整验证版](https://blog.csdn.net/zd199218/article/details/80698192)

