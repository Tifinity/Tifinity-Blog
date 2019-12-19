---
title: "服务计算第九周作业：博客网站API设计"
date: 2019-12-18T11:48:06+08:00
draft: false
tags: ["Api"] 
categories: ["服务计算"]             
author: "Tifinity"  
---

# 服务计算第九周作业：博客网站API设计
## 作业要求
规范：REST API 设计 [Github API v3 overview](https://developer.github.com/v3/) ；[微软](https://docs.microsoft.com/zh-cn/azure/architecture/best-practices/api-design)
作业：模仿 Github，设计一个博客网站的 API


## API设计
查看网站主页 GET "https://myblog"

- Request

  - Header

    ~~~json
    Authorization: token
    ~~~

- Response 200( application/json )

  ~~~json
  {
        "ok": true,
        "data": ""
  }
  ~~~



当前用户 GET "https://myblog/user"

- Request

  - Header

    ~~~json
    Authorization: token
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": {
          "id": "用户ID",
          "name": "用户",
      }
  }
  ~~~



用户身份验证 POST https://myblog.com/settings/connections/applications{/client_id}



发布文章 POST https://myblog/blog_edit

- Request

  - Header

    ~~~json
    Authorization: token
    ~~~

  - Body

    ~~~
     {
         "name": "文章名",
         "body": "文章内容"
     }
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": ""
  }
  ~~~



查看用户主页 GET https://myblog/users/{username}

- Request

  - Header

    ~~~json
    Authorization: token
    ~~~

- Response 200( application/json )

  ~~~json
  {
        "ok": true,
        "data": [
            {
                "id": 1,
                "name": "文章名",
                "link": "博客地址"
            }
        ]
  }
  
  ~~~



查看用户博客列表 GET https://myblog/users/{username}/blogs

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
        "ok": true,
        "data": [
            {
                "id": 1,
                "name": "文章名",
                "link": "博客地址"
            }
        ]
  }
  
  ~~~

  

查看用户个人信息 GET https://myblog/users/{username}/profile

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
        "ok": true,
        "data": {
            "id": "用户ID",
            "name": "用户名",
            "...": "..."
        }
  }
  
  ~~~



查看用户粉丝列表 GET https://myblog/users/{username}/fans

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": [
          {
              "name": "用户",
          }
      ]
  }
  
  ~~~



查看用户关注列表 GET https://myblog/users/{username}/followers

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": [
          {
              "name": "用户名"
          }
      ]
  }
  
  ~~~



关注用户 POST https://myblog/users/{username}/following

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": ""
  }
  
  ~~~



查看博客 GET https://myblog/blogs/{blogID}

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": {
          "name": "文章名",
          "link": "博客地址"
      }
  }
  
  ~~~



查看评论 GET https://myblog/blogs/{blogID}/comments

- Request

  - Header

    ~~~json
    Authorization: token
    
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": {
          "id": 1,
          "name": "文章名",
          "link": "博客地址"
      }
  }
  ~~~



发表评论 POST https://myblog/blogs/{blogID}/comments

- Request

  - Header

    ~~~json
    Authorization: token
    ~~~

  - Body

    ~~~json
    {
    	"comment"： "评论内容"
    }
    ~~~

- Response 200( application/json )

  ~~~json
  {
      "ok": true,
      "data": ""
  }
  ~~~