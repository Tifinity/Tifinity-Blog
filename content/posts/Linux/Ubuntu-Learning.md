---
title: "Ubuntu学习"
date: 2019-12-16T19:00:20+08:00
draft: false
categories: ["Linux"]
tags: ["Ubuntu"]
---

# Ubuntu学习

## 1. Ubuntu包管理机制

~~~shell
apt-get install <package>
apt-get update 
apt-get upgrade
~~~

这些命令我们都不会陌生，与windows不同，ubuntu的软件安装一般都是借助包管理工具在终端中实现。

如果你想知道apt-get到底是什么；

install的软件到底安装到哪去了；

update和upgrade的区别；

下载速度慢的时候换源是什么操作；

可以接着往下看本文。



### 1.1 软件包仓库

当你使用`apt-get install <package>`时，apt先到`/etc/apt`目录下去查找所有的仓库。这个目录下包含源仓库文件source.list和后缀为.gpg的gpg密钥文件。

![image-20191201134401534](https://github.com/Tifinity/MyImage/raw/master/GoOnlineReport/image-20191201134401534.png)

source.list长这样：

~~~shell
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ bionic main restricted
# deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ bionic-updates main restricted
# deb-src http://cn.archive.ubuntu.com/ubuntu/ bionic-updates main restricted
~~~

每一个仓库表示为：

~~~shell
deb/deb-src  <url>  <ubuntu版本代号>  <仓库类型>
~~~

其中

- deb-src表示指向源代码的仓库，deb指向软件仓库。

- url是仓库的域名

- ubuntu版本代号：

  - Xenial 16.04 (LTS)

  - Bionic 18.04 (LTS)

  - Cosmic 18.10

  - Disco 19.04
  - 等等

  若是bionic-updates表示更新仓库，执行`apt-get update`的时候使用该仓库。

- 最后一个部分表示仓库不同类型，可选项为：main，restricted，universe，multiverse

  - main : 表示由Canonical公司提供支持的免费开源的软件
  - universe : 表示由Ubuntu社区提供支持的免费开源的软件
  - restricted : 具有知识产权的设备专有驱动，比如英伟达自己开发的闭源驱动
  - multiverse : 受版权和法律保护的软件

然后我们直接访问仓库地址看看。

![image-20191201131728782](https://github.com/Tifinity/MyImage/raw/master/GoOnlineReport/image-20191201131728782.png)

dists/ 软件配置信息

pool/ .deb安装包

除了官方仓库，第三方仓库放在`etc/apt/sources.list.d/`目录下

![image-20191201133904920](https://github.com/Tifinity/MyImage/raw/master/GoOnlineReport/image-20191201133904920.png)

### 1.2 包管理工具

~~~shell
dpkg      – Debian 包安装工具
apt-get （ 现在用apt即可 ） – APT 的命令行前端
aptitude  – APT 的高级的字符和命令行前端
synaptic  – 图形界面的 APT 前端
dselect   – 使用菜单界面的包管理工具
tasksel   – Task 安装工具
~~~

可以先使用`dpkg --help`和`apt --help`查看帮助信息。

apt下载的包保存在`/var/cache/apt/archives`

未完待续

.deb 和 .rpm



## 2.修改终端提示信息

修改/etc/profile：

![image-20191227124252968](D:\TH\Tifinity-Blog\Tifinity\content\posts\image-20191227124252968.png)





## 参考资料

[UbuntuManual](https://wiki.ubuntu.org.cn/UbuntuManual:Ubuntu_%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%AE%A1%E7%90%86)

https://www.cnblogs.com/dennis-wong/p/9378145.html