---
title: "服务计算第八周作业：cloudgo"
date: 2019-12-18T11:37:14+08:00
draft: false
tags: [] 
categories: ["服务计算"]             
author: "Tifinity"  
---

# 处理 web 程序的输入与输出

## 文章目录

@[toc]
## 概述

设计一个 web 小应用，展示静态文件服务、js 请求支持、模板输出、表单处理、Filter 中间件设计等方面的能力。（不需要数据库支持）

## 任务要求

编程 web 应用程序 cloudgo-io。 请在项目 README.MD 给出完成任务的证据！

**基本要求**

1. 支持静态文件服务
2. 支持简单 js 访问
3. 提交表单，并输出一个表格
4. 对 `/unknown` 给出开发中的提示，返回码 `5xx`



## 完成基本要求

### 1. 框架选择

首先需要选择一个golang框架来进行开发，看了许多前辈的博客都说iris好，我想既然都没使用过那就随便选一个吧·~·

目前网上关于iris的教程比较少，主要参考[前辈博客](https://blog.csdn.net/Wonderful_sky/article/details/84036366)和[官方文档]( https://studyiris.com/doc/ )。

官方网站给出的安装指令：

~~~
go get -u github.com/kataras/iris
~~~

但是因为部分依赖包被墙了，是无法安装成功的，下面给出翻墙之外的方法。

先找到安装不成功的包，通过在get命令中使用-v参数看到安装的详细过程，看哪个包failed了，或者直接在网上随便找一个iris的程序`go run test.go`，看报错信息缺少哪个包。

~~~go
/*test.go*/
package main

import "github.com/kataras/iris"

func main() {
    app := iris.Default()
    app.Handle("GET", "/", func(ctx iris.Context) {
        ctx.HTML("Hello world!")
    })
    app.Run(iris.Addr(":8080"))
}
~~~

一般golang.org的包是无法访问的，但好在github上都有资源，所以只需要在github上搜索缺少的包手动`git clone`到golang.org/x目录(没有就自己创建)下即可。

![](https://img-blog.csdnimg.cn/20191112104819591.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

除此之外github.com下的包也可能缺失，删掉重新go get即可。

然后又出现错误：error未定义，翻看iris源码发现使用了error包，而error包似乎是golang 1.13才有的，于是更新golang到1.13。

运行`go run test.go`成功会看到命令行的提示信息。



### 2. 使用iris框架完成本次作业

#### 2.1 基本步骤

我的main.go如下：

~~~go
package main

import (
	"github.com/Tifinity/cloudgo/services"
	"github.com/kataras/iris"
)

func main() {
    // 获取app
	app := iris.Default()
	// 日志等级
	app.Logger().SetLevel("debug")
	// 运行
	services.StartServices(app)
	// 程序内配置服务端
	app.Run(iris.Addr(":8080"), iris.WithConfiguration(iris.Configuration{
	    DisableInterruptHandler:           false,
	    DisablePathCorrection:             false,

~~~

先获取app，通过`iris.Default()`获取默认app或者`iris.New()`新建一个app都可以。

~~~go
// 获取app
app := iris.Default()
~~~

然后可以设置日志等级，默认是INFO，DEBUG会在命令行窗口输出更详细的信息。

~~~go
// 日志等级
app.Logger().SetLevel("debug")
~~~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112104842509.jpg)

然后是运行端口和服务器配置，可以在程序内通过代码配置，也可以通过TOML，YAML文件配置，配置项是抄官方文档的，并且app初始化时已经使用了默认的配置值，无需配置也可以启动我们的程序。

~~~go
// 程序内配置服务端并运行
app.Run(iris.Addr(":8080"), iris.WithConfiguration(iris.Configuration{
    DisableInterruptHandler:           false,
    DisablePathCorrection:             false,
    EnablePathEscape:                  false,
    FireMethodNotAllowed:              false,
    DisableBodyConsumptionOnUnmarshal: false,
    DisableAutoFireStatusCode:         false,
    TimeFormat:                        "Mon, 02 Jan 2006 15:04:05 GMT",
    Charset:                           "UTF-8",
}))
~~~



#### 2.2 支持静态文件服务

只需一行代码即可支持静态文件的传输，不过前辈博客中的StaticWeb方法已经不能用了，官方文档也还是使用的StaticWeb。

~~~go
//app.StaticWeb("/static", "./static")
app.HandleDir("/static", "./static")
~~~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112104853739.jpg)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112104901814.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
图片来源于B站，[·~·](https://space.bilibili.com/3990478/)

#### 2.3 支持简单 js 访问

简单js访问，比如点击一下按钮跳转到另一个页面。

~~~js
/*test.js*/
// 新建窗口打开/static
function f() {
    window.open("/static/img/test.webp")
}

/*hello.html*/
<input type="button" onclick="f()" value="js">
~~~



#### 2.4 提交表单，并输出一个表格

使用模板显示视图，需要先注册，两个参数分别是文件夹和文件类型，Reload(true)表示html更新时不用重新启动app。

~~~go
app.RegisterView(iris.HTML("./templates", ".html").Reload(true))
~~~

在hello.html中定义表单，请求方法为POST。

~~~html
<form method="POST" action="/info">
	say something: <input type="text" name="Something"><br/><br/>
	<input type="submit" value="submit"><br/><br/>
</form>
~~~

首先定义一个表单结构体，字段名称需要和html中的表单字段名相同（在这里是Something）， 首字母必须大写，使用ReadForm将表单读取到form中，使用ViewData填充返回的html文件，第一个参数是key，第二个是value。

~~~go
func StartServices(app *iris.Application) {
    /*···*/
	app.Post("/info", getInfo)
    /*···*/
}

func getInfo(ctx iris.Context) {
	form := Form{}
	err := ctx.ReadForm(&form)
	if err != nil {
		ctx.StatusCode(iris.StatusInternalServerError)
		ctx.JSON(iris.Map{
			"error": "Something Wrong",
		})
	} else {
		ctx.ViewData("something", form.Something)
		ctx.View("info.html")
	}
}
~~~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112104917600.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112104923744.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

#### 2.5 对 `/unknown` 给出开发中的提示，返回码 `5xx`

使用StatusCode设置错误码，然后通过WriteString或者JSON返回错误信息。

~~~go
func unknown(ctx iris.Context) {
	ctx.StatusCode(501)
	ctx.JSON(iris.Map{
		"error": "501 ~开发中~",
	})
}
~~~

同时在特定错误发生时可以自定义处理函数，比如：

~~~go
func StartServices(app *iris.Application) {
    /*···*/
	app.OnErrorCode(iris.StatusNotFound, notFound)
    /*···*/
}

func notFound(ctx iris.Context) {
    ctx.JSON(iris.Map{
		"error": "404 Not Found",
	})
    /*可以返回JSON，可以返回html等等*/
}
~~~

用postman发出请求，方便看到状态码（右边绿色）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191112105017430.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

## 其他
[github地址](https://github.com/Tifinity/HelloGo/tree/master/cloudgo)

### 参考资料

[官方英文文档](https://docs.iris-go.com/iris/ )

[官方中文文档]( https://studyiris.com/doc/ )

[前辈博客](https://blog.csdn.net/Wonderful_sky/article/details/84036366)