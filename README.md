---
title: "About"
date: 2019-12-15T17:52:04+08:00
draft: false
---


# Tifinity-Blog

这是一个基于Hugo，部署在GitPage的个人博客网站。这里记录博客站的搭建，维护过程。

## 如何使用

选择一个静态博客的生成工具，因为现在用 Go，所以选择了同样是 Go 写的 [Hugo](https://www.gohugo.org/)。

~~~bash
# 通过 brew 安装 hugo
brew install hugo
# 查看版本
hugo version
hugo v0.87.0+extended darwin/arm64 BuildDate=unknown
# 创建新站点
hugo new site demo-site
~~~

新创建的站点目录结构如下：

~~~bash
demo-blog					
├── archetypes			# 原型文件，md文件的模版
│   └── default.md	
├── config.toml			# 配置文件
├── content					# 文章存放于此
├── data						#	
├── layouts					#	
├── static					#	静态资源
└── themes					# 主题
~~~

现在用 hugo 创建一篇文章，与直接创建 md 文件的唯一区别就是增加了头部的注释。

~~~bash
# 创建一篇文章，会自动在当前目录的content下创建
cd demo-blog && hugo new about.md
<path>/demo-blog/content/about.md created
~~~

新文章的头部注释是根据 archetypes 中的 default.md 生成的，包括创建时间，标题，是否是草稿（草稿不会显示在网页上）。更换主题之后原型文件也要更换，在头部注释中可能包含类别，标签，作者等其他信息。

~~~bash
# 用官方主题库替换站点文件夹的 themes
git clone --depth 1 --recursive https://github.com/spf13/hugoThemes.git themes
~~~

## 感激不尽

[wu-kan](https://wu-kan.github.io/) 让我第一次接触个人博客站，了解到GithubPages，感谢。

[leopardpan](http://baixin.io:8000/2016/10/jekyll_tutorials1/) 早期的博客模板以及教程，感谢。

[Github Pages](https://pages.github.com/) 提供个人网站服务，感谢。

[Hugo]() 提供网站框架，感谢。

[LeaveIt by 柳志超](https://themes.gohugo.io/leaveit/) 提供博客主题，感谢。

[Valine](https://valine.js.org/)和[LeanCloud](https://leancloud.cn/) 提供评论系统服务，感谢

## 长路漫漫

### NOW

- [x] 还在起步

- [ ] 正在进行博客的搬运工作（持续进行中）

- [x] 制作网站图标[Favicon](https://realfavicongenerator.net/)

- [ ] 放自己的背景图

- [ ] 文章中图片与文字对齐


### 2019-12-19

- 确定主题为 柳志超先生 的 [LeaveIt](https://themes.gohugo.io/leaveit/) 
- 加入[Valine](https://valine.js.org/quickstart.html)评论系统，感谢[hugo LeaveIt主题优化一](https://www.jianshu.com/p/d8f0c924bc3a)
- 加入不蒜子流量统计

### 2019-12-12

> 人生没有白走的路，每一步都算数。	——李宗盛

- 重新开始

- 基于Hugo搭建博客站

  写作应该是一个相对单纯的事情，使用Hugo的人应该更专注于写作。

- 使用比较笨的方法自动部署

  把add，commit，push等写在脚本里强制push。

### 2019-04-26

- 从[wu-kan](https://wu-kan.github.io/)的教程快速启动，搭建基于Jekyll的博客站


## 愚者自知

本人写博客的主要作用还是记录自己的学习过程，有缘人观之，同路者共勉。
