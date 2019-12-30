---
title: "Scoop-为Windows而生的命令行包管理工具"
date: 2019-12-27T19:43:39+08:00
draft: false
categories: ["工欲善其事，必先利其器"]
tags: ["Scoop","包管理"]
featured_image: 
author: "Tifinity"
---

# Scoop第一次使用过程

## 为什么要使用包管理工具？

首先，在Windows下我们一般怎么安装软件？:anger:

1. 百度你要安装的软件
2. 找到官网或者一个看起来安全一点的地址
3. 下载一个奇怪的exe
4. 安装到奇怪的地方

仔细想想，我从用Windows开始一直以来都是这样安装软件的，直到开始用Linux的操作系统才对**包管理**有了模糊的概念，不过也只是停留在“Linux跟Windows安装软件的方法有一点不一样”的阶段。

>包管理器又称**软件包管理系统**，它是在电脑中自动安装、配制、卸载和升级[软件包](https://baike.baidu.com/item/软件包)的工具组合，在各种[系统软件](https://baike.baidu.com/item/系统软件)和[应用软件](https://baike.baidu.com/item/应用软件)的安装管理中均有广泛应用。——百度百科

我对Windows电脑软件的管理经历了三个阶段：

1. 腾讯电脑管家
2. 手动管理+Windows自带的卸载
3. 发现了Scoop，开始了解包管理工具

Scoop的官方文档上这么写着：

Scoop从命令行以最小的摩擦安装程序。 它试图消除类似的东西：

- 权限弹出窗口
- GUI向导式安装程序
- 安装大量程序造成的路径污染
- 安装和卸载程序产生的意外副作用
- 查找和安装依赖项的需要
- 需要执行额外的设置步骤才能获得有效的程序

## 安装配置

首先需要几个环境要求：

- Windows 版本不低于 Windows 7
- PowerShell  3+
- .NET Framework 4.5+
- 用户名无中文

如果PowerShell和.NET不满足版本要求的话还是得先用百度更新。:broken_heart:

~~~powershell
# 允许本地脚本执行
set-executionpolicy remotesigned -scope currentuser
# 下载安装scoop
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
~~~

现在等待脚本执行完成。:busstop:

安装完成后请迫不及待地执行`scoop help`，看到帮助信息说明安装成功了。

![](https://github.com/Tifinity/MyImage/raw/master/Tools/Scoop/1577448318(1).jpg)

## 简单使用

你已经看过了help输出的使用说明，如果以后某个指令想不起来也请执行`scoop help`。

| 常用命令  | 含义         |
| --------- | ------------ |
| search    | 搜索软件名   |
| install   | 安装软件     |
| update    | 更新软件     |
| status    | 查看软件状态 |
| uninstall | 卸载软件     |
| info      | 查看软件详情 |
| home      | 打开软件主页 |

举个:chestnut:：

我要安装Typora，先搜索一下`scoop search typora`

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/Scoop/20191227200830.png)

bucket可以简单理解为仓库，看信息的意思是在一个“其他”仓库中，其实'extras'仓库是官方维护的扩展仓库，因为默认仓库'main'中软件很少，所以我们先添加'extras'：

~~~powershell
scoop bucket add extras https://github.com/lukesampson/scoop-extras
~~~

如果报错保证自己能访问这个地址，再重试，还不行再百度。

但是现在发生了一件很奇怪的事情，我再搜索typora，没有了。:grey_question::grey_question:

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/Scoop/20191228101213.png)

随后到[Github](https://github.com/lukesampson/scoop-extras/tree/master/bucket)的extras仓库查找，确实存在。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/Scoop/20191228101123.png)

查看帮助`scoop help bucket`，有了发现

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/Scoop/20191228101923.png)

回头看看上面的指令，url后面没有.git，这真是跟main一样的错误。

删了重新添加之后一切正常，这里的bucket需要的是git仓库地址，如果url里没有.git就只是可以通过浏览器访问的项目地址而已。

现在可以通过`scoop install typora`安装typora了。

现在打开C:\Users\TIFINITY\scoop，Scoop在这里创建了一个scoop文件夹，Scoop把软件安装在apps中。

## 还需要解决的问题

下载速度比较慢，因为都是从官网下载，不过可以使用arias2来加速。

Scoop适合安装开源的，比较小的软件，或者是用来配置开发环境，如果是大型软件还是走官方网站⑧:cowboy_hat_face:

## 参考资料

[Scoop官网](http://scoop.sh/)

[Scoop Github](https://github.com/lukesampson/scoop)

[Scoop Wiki](https://github.com/lukesampson/scoop/wiki)

[少数派： 「一行代码」搞定软件安装卸载，用 Scoop 管理你的 Windows 软件](https://sspai.com/post/52496)