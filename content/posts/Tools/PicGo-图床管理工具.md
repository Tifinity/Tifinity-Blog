---
title: "PicGo-图床管理工具"
date: 2019-12-27T17:53:45+08:00
draft: false
categories: ["工欲善其事，必先利其器"]
tags: ["PicGo"]
featured_image: https://raw.githubusercontent.com/Tifinity/MyImage/master/Logo/PicGo.png
author: "Tifinity"
---

# PicGo+Github

PicGo是一个非常好用的开源图床管理工具：[Github地址](https://github.com/Molunerfinn/PicGo)

我的图床目前是Github，下面就以Github来进行一遍操作。

在Github上创建一个仓库用来保存图片，然后创建一个token，点击头像->Setting->Developer setting->Personal access tokens->Generate new token，填写Note，勾选repo，复制保存token，**这个 token 只显示一次**。

然后打开PicGo->图床设置->Github设置：

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/image-20191227180516666.png)

在PicGo设置中开启上传前重命名，可以手动修改图片名，更好的分类管理图片，PicGo会在仓库中按路径存放图片，文件夹不存在会自动创建。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/image-20191227181431021.png)

在Github仓库中能够看到上传的图片。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/image-20191227181611880.png)

从此告别了写一篇博客还要同步操作图床的麻烦。

