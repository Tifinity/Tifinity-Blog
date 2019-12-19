---
title: "服务计算第十二周作业：简单 web 服务与客户端开发实战"
date: 2019-12-18T11:11:38+08:00
draft: false
tags: [] 
categories: ["服务计算"]             
author: "Tifinity"  
---

# 服务计算作业：简单 web 服务与客户端开发实战

## 作业要求概述

利用 web 客户端调用远端服务是服务开发本实验的重要内容。其中，要点建立 API First 的开发理念，实现前后端分离，使得团队协作变得更有效率。

[作业要求](https://pmlpml.github.io/ServiceComputingOnCloud/ex-services)

## 开发过程

本次作业我们选择实现见单个人博客网站：[项目地址](https://github.com/Simple-Blog)

以下是本人承担的工作.

### 使用Github建立组织

在github创建本次作业的组织Simple-Blog，创建三个仓库：

- backend：后端
- frontend：前端
- docs：项目文档

邀请所有组员加入，并在自己的分支下分别进行开发，本地测试无Bug后再发起Marge Request，由创建者Marge。

参考：[建立组织](https://chun-ge.github.io/How-to-establish-an-organization-on-Github/)

### 设计API

~~~json
{
    "ArticlePost": "/openapi101/users/{username}/article",
   	"ArticleArticleIdGet":"/openapi101/users/{username}/article/{article_id}",
    "CreateComment":"/openapi101/users/{username}/article/{article_id}/comment",
    "GetCommentsOfArticle":"/openapi101/users/{username}/article/{article_id}/comment",
    "AuthSigninPost":"/openapi101/auth/signin",
    "AuthSignupPost": "/openapi101/auth/signup"
}
~~~

具体api设计见项目文档中的api.yaml

[Github API v3 overview](https://developer.github.com/v3/)

### 使用API工具生成框架

使用swagger，swagger是一个简单强大的API表达工具，只需要在[在线编辑器](http://editor.swagger.io/)上用YAML或者JSON写好API，就可以自动生成几乎所有语言框架的API。

网页左边是编辑器，右边是实时响应的预览界面。

![image-20191209231219892](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw10/image-20191209231219892.png)

部分api如下：

~~~yaml
/user/{username}/article/{article_id}:
    get:
      summary: get post
      parameters:
        - name: username
          in: path
          required: true
          description: user's name
          type: string

        - name: article_id
          in: path
          required: true
          description: article id
          type: integer
          
      responses:
        200:
          description: A post
          schema:
            $ref: '#/definitions/Article'
        404:
          description: Not Found
~~~

swagger的使用教程与yaml编写api的学习参考:[Swagger Gitbook](https://huangwenchao.gitbooks.io/swagger/content/)



### 后端开发

#### 生成框架

在编辑器上方菜单栏点击GenerateServer->go server即可生成golang的后端框架，项目结构如下：

go-server-server

- .swagger-codegen
- api
  - swagger.yaml：api设计文件
- go：
  - logger.go：日志
  - routers.go：路由匹配
  - model_(*).go：资源模型
  - api_default.go：api框架
- .swagger-codegen-ignore
- main.go：项目入口，主文件

项目通过main.go进入，收到请求时通过routers.go匹配api_default.go中对应的函数。既然框架已经搭好，我们就只需要关心逻辑实现了。

#### 实现注册

1. 通过json反序列化http请求的body，得到username和password，判断是否符合规则，不符合返回错误信息，符合进行下一步；
2. 查找数据库判断有无重复用户，有则返回错误信息，无则进行下一步；
3. 更新数据库；
4. 响应200.

#### 实现登录

1. 通过json反序列化http请求的body，得到username和password，判断是否符合规则，不符合返回错误信息，符合进行下一步；
2. 查找数据库，判断是否有该用户，密码是否正确；
3. 返回JWT；
4. 响应200.

#### 关于boltdb数据库的使用：

开启数据库:

~~~go
db, err := bolt.Open("my.db", 0600, nil)
~~~

通过View进行只读事务，通过User桶查找用户：

```go
err = db.View(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte("User"))
    if b != nil {
        v := b.Get([]byte(user.Username))
        if v != nil {
            return errors.New("User Exists")
        }
    }
    return nil
})
```

通过Update进行写操作，如果没有User桶则先创建，然后使用Put写入用户名和密码的键值对。

需要注意的是所有写入操作都需要写入[]byte。

boltdb数据库学习：[golang boltdb的学习和实践](https://segmentfault.com/a/1190000010098668)

#### 关于JWT验证

JWT（Json Web Token）是一种验证用户身份的方式，一般使用方式为：用户登录后，服务端发给客户端一个token，客户端将其存放在cookie中，之后客户端与服务端的交互都需要在请求的Header中加上Authorization参数，值即为该token。

这个token的组成为：

JWT头：

{

	"alg": "HS256",
	
	"typ": "JWT"

}

"alg"表示签名算法为HS256，"typ"表示token类型为JWT。

有效载荷：

{

	iss：发行人
	
	exp：到期时间
	
	sub：主题
	
	aud：用户
	
	nbf：在此之前不可用
	
	iat：发布时间
	
	jti：JWT ID用于标识该JWT

}

除了这些默认可选字段之外也可以自己定义字段。

签名：

用来保证JWT的真实性。

JWT生成与分发：

~~~go
token := jwt.New(jwt.SigningMethodHS256)
claims := make(jwt.MapClaims)
claims["exp"] = time.Now().Add(time.Hour * time.Duration(1)).Unix()
claims["iat"] = time.Now().Unix()
claims["sub"] = user.Username
token.Claims = claims

tokenString, err := token.SignedString([]byte(user.Username))
if err != nil {
    w.WriteHeader(http.StatusInternalServerError)
    fmt.Fprintln(w, "Token error")
    log.Fatal(err)
}
~~~



[Go实战--golang中使用JWT(JSON Web Token)](https://blog.csdn.net/wangshubo1989/article/details/74529333)

[10分钟了解JSON Web令牌（JWT）](https://baijiahao.baidu.com/s?id=1608021814182894637&wfr=spider&for=pc)

#### 后端测试

后端使用Postman或者Postwoman发送不同的请求来进行接口的测试。

![image-20191210001314114](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw10/image-20191210001314114.png)

![image-20191210001402677](https://github.com/Tifinity/MyImage/raw/master/ServiceComputing/hw10/image-20191210001402677.png)