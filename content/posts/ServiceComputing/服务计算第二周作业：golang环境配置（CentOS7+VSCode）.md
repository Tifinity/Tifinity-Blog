---
title: "服务计算第二周作业：golang环境配置（CentOS7+VSCode）"
date: 2019-12-18T11:14:56+08:00
draft: false
categories: ["服务计算"]             
author: "Tifinity"  
---

# 服务计算实验二：golang环境配置（CentOS7+VSCode）
[github项目](https://github.com/Tifinity/HelloGo)
### Linux文件目录结构

在安装golang之前，为了避免许多不必要的错误，一定要分清root用户和你使用图形界面的用户，即你自己取了个名字的用户，以下为了方便称为**非root用户**。

进入root用户的命令为`su`，退出是按Ctrl + D。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915131933792.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
/root就是root用户，系统管理员，超级权限者的主目录，而**非root用户**的主文件夹是在**/home**下的。

`pwd`：查看当前位置

> **/usr**：这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。
>
> **/etc：**
> 这个目录用来存放所有的系统管理所需要的配置文件和子目录。

详细请看[菜鸟教程][runoob]

### golang环境配置-Linux

安装包下载地址：[GO语言中文网][go-zh]  或者  [RPM包下载][rpm]

Windows直接运行`.msi`文件即可。

Linux下

- 如果下载的是`.tar`文件，进入文件目录使用命令`tar [压缩包文件名]`解压，你需要记住解压的位置。如果提示没有权限则在前面加上`sudo`再运行。
- 如果下载的是`.rpm`文件，进入文件目录使用命令`rpm -ivh [文件名]`会自动安装golang到该目录：`/usr/lib/golang`，可参见上图Linux文件目录结构。

推荐新手使用`.rpm`的方式安装，安装目录固定并且会自动设置环境变量，不容易出错。如果想详细了解`.tar.gz`和`.rpm`文件的区别，请前往 -> [?][rpm-tar]。

[外链图片转存失败(img-OA5sNPSK-1568524694199)(T:\TH\大三上\服务计算\3\image\安装.jpg)]

安装完成后执行`go version`查看版本，有类似下图输出则安装成功。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132030653.jpg)

### 安装vscode

1. 可以使用snap安装

   `sudo snap install --classic code`

2. 下载.rpm文件安装，如果安装过程中需要什么依赖就到此处搜索下载：[?][rpm]

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132043423.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

   安装命令同go的rpm安装，参看上文。安装完成后可在左上角应用程序->活动概览找到VSCode

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132051792.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
### 工作空间

就是自己创建的文件夹，应该被创建成这样：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132209221.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

一开始只要有bin，pkg，src就行了，src用来放后缀是**.go**的源代码，就想**.cpp**或**.py**，pkg放编译过的包以便再次运行时节约时间，bin用来放可执行文件。每一次编写go语言的项目应该在一个工作空间中进行。

### 环境变量

目前我们需要用到的环境变量只有两个，一个是GOPATH，一个是PATH。

修改环境变量有几种方式：

- 临时：

```
export PATH=$PATH:$GOPATH/bin
export GOPATH=$HOME/work
```

- 永久：需要修改文件，进入对应目录并使用`vim [文件名]`打开配置文件，把上面的语句添加到文件中。
  - 修改`/home/[用户名]/.bashrc`，隐藏文件需要使用`ls -a`才能看到，该方法对单一用户生效。**本次建议使用该方法**
  - 修改`/etc/profile`，对所有用户生效，需要root权限。

详细设置可以参看 -> [?][env]

HOME也是一个环境变量，可以在终端中使用` env`命令查看所有环境变量，一般为用户的主文件夹。同时环境变量之间也是可以互相嵌套的。

PATH中包含一些路径，当你执行可执行程序时终端会自动在PATH中包含的路径中寻找你执行的程序，如果找到则执行。PATH一般有多个值，指令中冒号的意思是在原来的PATH上添加冒号后的值。此处设置PATH可以让你在任何地方执行你的go可执行程序，即hello。

GOPATH指定当前工作空间的位置，在使用go命令的时候会自动到GOPATH指定的工作空间中寻找需要的文件，比如你可以在任何位置使用该语句：

```
go install github.com/user/hello
```

go工具会自动在路径前加上$GOPATH/，即/root/gowork/

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019091513222884.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

你可以使用 `echo $PATH` 来查看环境变量的值，也可以使用 `env` 来查看所有环境变量的值。

或者 `go env` 可以查看关于go的一下变量：

- `GOROOT` 是go安装的目录，须知不能将工作空间与安装目录设置在同一目录。
- `GOBIN` 是可执行文件的目录
- `GOPATH` 是工作空间的目录

### 文件带锁

当你的文件出现有一个小锁的图标时，意味着该文件属于别的用户，比如root，你想要通过图形界面的用户来使用的话，需要权限。

解锁的过程：

```
sudo chown -P [用户名] [路径：将文件或文件夹拖到此处即可]
```

参数`-P`的意思是将该文件及其子文件都进行同样的操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132313844.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

同样也可以加锁，即转移控制权给root

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132323910.jpg)

### 创建第一个go源文件

现在可以开始写go程序了。

先按照官方文档[如何使用GO编程][go-teach]，将工作空间创建成这样：

```
bin/
pkg/
src/
	github.com/
		github-user/ /*官方文档中叫user，老师的github中叫github-user，都行*/
			hello/
```

然后使用`go install github.com/github-user/hello`编译并安装你的hello.go

- 如果没有任何输出说明没有问题，你可以在`bin/`里看到名为hello的可执行程序。这时你可以在任何地方执行hello命令，比如我`/usr/lib/bin`里执行hello。（这里做了后面步骤所以有个反向的）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915132949556.jpg)
- 如果输出没有找到文件的话，需要检查你的环境变量是否设置正确。

在文件目录下执行`go run hello.go`可以终端窗口看到输出。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915133007159.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

在文件目录下执行`go build hello.go`可以在当前目录生成hello可执行程序，执行`.\hello`同样可以看到输出。

在任何地方执行`go install github.com/github-user/hello`可以构建并安装你的程序，可以在bin文件夹中看到hello可执行程序，如果将bin文件加加入PATH环境变量即可在任何地方执行hello运行该程序。



### 创建第一个库

按照同样的方式创建recerse.go。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915134347338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

将hello.go修改为

```go
package main

import (
	"fmt"
	"github.com/github-user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("hello, world"))
	fmt.Printf("\nhello, world\n")
}

```

再次安装hello，`go install github.com/github-user/hello`，运行可看到效果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190915134355726.jpg)


### 第一个测试文件
按照教程写好测试文件，并执行`go test github.com/github-user/stringutil`，结果如下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916100805972.png)

[1]: https://www.cnblogs.com/-Fly/p/11101978.html
[runoob]: https://www.runoob.com/linux/linux-system-contents.html
[go-zh]: https://studygolang.com/dl
[rpm]: http://rpmfind.net/linux/rpm2html/search.php?query=golang(x86-64)
[rpm-tar]: https://www.cnblogs.com/zhuawang/p/4927344.html
[go-teach]: https://go-zh.org/doc/code.html
[env]: https://www.cnblogs.com/qiuhong10/p/7815943.html