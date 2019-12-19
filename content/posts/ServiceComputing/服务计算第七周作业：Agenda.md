---
title: "服务计算第七周作业：Agenda"
date: 2019-12-18T11:46:25+08:00
draft: false
tags: [] 
categories: ["服务计算"]             
author: "Tifinity"  
---

# 服务计算第七周作业：Agenda

## 文章目录

@[toc]
## 概述

> 

## 安装Cobra

cobra既是一个用来创建强大的现代CLI命令行的golang库，也是一个生成程序应用和命令行文件的程序。因为本次项目是一个命令行程序，所以涉及到读写参数问题，之前是使用pflag来实现参数的读入。但是cobra的使用可以快速生成命令行文件程序，构建一个命令行程序的框架。 

首先安装被墙的依赖包。

在`$GOPATH/src/golang.org/x`目录下，若没有则自行创建，用`git clone`下载sys和text项目： 

```
git clone https://github.com/golang/sys
git clone https://github.com/golang/text
```

然后执行：

~~~
go get -v github.com/spf13/cobra/cobra
~~~

若成功安装则在 `$GOBIN` 即 `$GOPATH/bin` 下出现cobra可执行程序。
然后在命令行中输入`cobra`: 

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-cWEKKvcf-1571931179495)(image\cobra安装.jpg)]

出现图中提示信息则安装成功。



## 创建程序

使用` cobra init `命令初始化程序框架，但是发现提示错误：

~~~
Error: required flag(s) "pkg-name" not set
~~~

在查看[官方文档][cobragithub]后发现Cobra版本更新后需要增加一个必须参数`--pkg-name`，就是main函数默认import的包。

> Updates to the Cobra generator have now decoupled it from the GOPATH. As such `--pkg-name` is required. 

于是我们使用如下命令初始化：

1. 先创建目录进入再初始化，此时不需要[name]参数，即目录

~~~
mkdir -p newApp
cd newApp
cobra init --pkg-name github.com/github-user/newApp
~~~

2. 直接创建目录新目录

~~~
cobra init --pkg-name github.com/github-user/newApp newApp
~~~

初始化成功后出现如下提示信息：

~~~
Your Cobra applicaton is ready at <path>
~~~

此时项目结构应如下：

~~~
agenda/
    cmd/
    main.go
    LICENSE
~~~

main.go长这个样子：

~~~go
package main

import "github.com/github-user/agenda/cmd" //pkg-name

func main() {
  cmd.Execute()
}
~~~



cmd文件夹里放的就是我们接下来要添加的命令。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-lTspuo4l-1571931179497)(image\cobra初始化.jpg)]



## 添加命令

cobra添加命令的指令如下：

~~~
cobra add <conmand_name>

~~~

于是我们进入刚创建的目录下，执行：

~~~
cobra add register
cobra add login

~~~

cmd中会自动添加两个go文件：

~~~
agenda/
    cmd/
    	register.go
    	login.go
    main.go
    LICENSE

~~~



## 总结

本次作业在难度上比前两次要大很多。本质上来说是实现了一个“翻译”的任务，实现了golang版的selpg。大二操作系统也有写过C语言的命令行程序，所以对CLI也不是一窍不通，不过这一次对管道，重定向有了更深的了解，也简单使用了golang的命令行参数处理包flag和pflag，执行外部指令的exec，还有os和io的一些操作。过程还是比较坎坷，虽然网上参考的博客很多，但是鱼龙混杂，其中很多都有错的或者理解不到位的地方，自己真正会了才可能完整地做出来，否则即使正确的结果也只是简单的摆拍而已。

github地址 -> [?]()

项目中附测试截图

## 参考资料

[go语言子进程](https://shimo.im/sheets/KJXKY9YgchdKJXGx/MODOC/)

[go语言创建读取写入文件](https://studygolang.com/articles/14170)

[某博客](https://blog.csdn.net/C486C/article/details/82990187)

[某另一博客](https://blog.csdn.net/hcm_0079/article/details/82927996)

[开发Linux命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)

[selpg源代码](https://www.ibm.com/developerworks/cn/linux/shell/clutil/selpg.c)