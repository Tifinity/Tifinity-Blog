---
title: "PicGo-图床管理工具"
date: 2019-12-27T17:53:45+08:00
draft: false
categories: ["工欲善其事，必先利其器"]
tags: ["PicGo", "Github图片无法加载"]
featured_image: https://raw.githubusercontent.com/Tifinity/MyImage/master/Logo/PicGo.png
author: "Tifinity"
---

# PicGo+Github图片管理方案

### PicGo使用

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

### 修改hosts解决Github图片无法加载问题

如果出现在Github上图片加载不出来，PicGo的相册里也加载不出图片，像下面这样，一般是Github网站的问题。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/PicGo/1580456278(1).jpg)

第一步，用管理员权限打开文件，C:\Windows\System32\drivers\etc\hosts。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/PicGo/1580456790(1).jpg)

~~~
# 这两行是固定的，直接复制
192.30.253.112 github.com
192.30.253.119 gist.github.com

# 下面的每个人不一样，获取方法如下文
199.232.28.133 githubusercontent.com
199.232.28.133 assets-cdn.github.com
199.232.28.133 raw.githubusercontent.com
199.232.28.133 gist.githubusercontent.com
199.232.28.133 cloud.githubusercontent.com
199.232.28.133 camo.githubusercontent.com
~~~

第二步，打开[IP查询网站](https://www.ipaddress.com/)，从上面随便复制一个域名输入下图的查询框，点击查询。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/PicGo/1580457383.png)

得到你的IP地址，填入hosts里，保存，刷新Github网站，现在可以正确显示图片。

![](https://raw.githubusercontent.com/Tifinity/MyImage/master/Tools/PicGo/1580458347(1).jpg)