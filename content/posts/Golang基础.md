---
title: "Golang基础"
date: 2019-12-24T14:11:21+08:00
draft: true
categories: ["未分类"]
tags: ["Go"]
featured_image: 
author: "Tifinity"
---

# Golang基础

## 依赖管理

### GOPROXY

因为众所周知的原因Go语言的有些第三方包无法通过`go get `命令下载。有两个方法可以解决这个问题，一是手动到Github上`git clone`所需要的包，不过稍微麻烦一点，二是通过 GOPROXY 环境变量设置代理服务器，一步到位，再也不用担心被墙了。

目前公开的代理服务器的地址有：

- goproxy.io
- goproxy.cn（推荐使用，由七牛云提供）

Windows：

~~~shell
go env -w GOPROXY=https://goproxy.cn,direct
~~~

或者此电脑->属性->高级->环境变量->新建

MacOS 或 Linux：

~~~shell
# 在~/.profile中加上
export GOPROXY=https://goproxy.cn
# 再执行
source .profile
~~~



fmt.PrintIn()和fmt.Printf()的区别：

- fmt.PrintIn(num)，不支持格式化
- fmt.Printf("%d", num)，支持格式化

切片长度len和切片容量cap并不相等，len相当于现在有多少个元素，cap是现在可容纳多少个元素，超过cap就要扩容。

像切片头插入元素会导致元素被全部复制一遍，性能会差很多。

for range返回的是元素的副本，而不是直接引用。

#### 并发/并行

多线程程序在单核心的 cpu 上运行，称为并发；多线程程序在多核心的 cpu 上运行，称为并行。

