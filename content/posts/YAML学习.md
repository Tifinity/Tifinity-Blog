---
title: "YAML学习"
date: 2019-12-20T23:11:18+08:00
draft: false
categories: ["未分类"]
tags: ["YAML"]
author: "Tifinity"
---

# YAML学习

YAML 是 "YAML Ain't a Markup Language"（YAML 不是一种标记语言）的递归缩写。

在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。

YAML 语言（发音 /ˈjæməl/ ）的设计目标是方便人类读写。它实质上是一种通用的数据串行化格式。



### 基本语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进只允许空格
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示单行注释



### 数据结构

- 对象：键值对集合，又称映射/哈希/字典

  ~~~yaml
  # 冒号后的空格不可省略
  key: value
  key: {name: TH, money: Infinit}
  # 复合
  key:
      key1: value1
      key2: value2
  ~~~

  

- 数组：或序列，列表

  ~~~yaml
  - a
  - b
  - c
  ~~~

  

- 纯量（scalars）：单个不可分割的值，包括：

  - 字符串，布尔，整数，浮点数，Null，时间，日期

  ~~~yaml
  boolean: TRUE    #true，True都可以
  float: 6.8523015e+5  #可以使用科学计数法
  int: 0b1010_0111_0100_1010_1110    #二进制表示
  null: ~  #使用~表示null
  string:
      - 哈哈
      - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
      - newline
        newline2    #字符串可以拆成多行，每一行会被转化成一个空格
  date:
      - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
  datetime: 
      - 2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
  ~~~

  两个感叹号`!!`可以强制转换类型。



### 字符串

单引号

双引号不会对特殊字符转义

~~~yaml
s1: '内容\n字符串'
s2: "内容\n字符串"
~~~

转换成JS如下：

~~~js
 { s1: '内容\\n字符串', s2: '内容\n字符串' }
~~~

多行字符串使用`|`保留换行符，`>`折叠换行。



### 引用

锚点`&`和别名`*`，可以用来引用。

~~~yaml
defaults: &defaults
    adapter:  postgres
    host:     localhost

development:
    database: myapp_development
    <<: *defaults

test:
    database: myapp_test
    <<: *defaults

 # 相当于下面
 defaults:
     adapter:  postgres
     host:     localhost

development:
    database: myapp_development
    adapter:  postgres
    host:     localhost

test:
    database: myapp_test
    adapter:  postgres
    host:     localhost
~~~



### 参考资料

[阮一峰](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

[菜鸟教程](https://www.runoob.com/w3cnote/yaml-intro.html)