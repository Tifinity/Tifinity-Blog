---
title: "About"
date: 2019-12-15T17:52:04+08:00
draft: false
---
# Tifinity-Blog

[![Deploy Hugo Blog](https://github.com/Tifinity/Tifinity-Blog/workflows/Deploy%20Hugo%20Blog/badge.svg)](https://github.com/Tifinity/Tifinity-Blog/actions)

https://tifinity.github.io/

我的个人博客，记录平时学习。

博客生成工具为 Hugo，博客托管发布于 GitHub Pages。

## 如何使用

选择一个静态博客的生成工具，选择 [Hugo](https://www.gohugo.org/)，没什么原因，随便选的。

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

## 仓库架构说明

本项目采用 **源码与部署分离** 的架构，涉及两个 GitHub 仓库：

### 仓库关系

| 仓库 | 用途 | 地址 |
|------|------|------|
| **Tifinity-Blog** | 源码仓库，包含 Markdown 文章、主题、配置 | 当前仓库 |
| **Tifinity.github.io** | 部署仓库，包含 Hugo 生成的静态文件 | [github.com/Tifinity/Tifinity.github.io](https://github.com/Tifinity/Tifinity.github.io) |

### 架构图解

```
Tifinity-Blog (源码)                    Tifinity.github.io (部署)
├─ content/posts/     ──Hugo构建──►     ├─ index.html
├─ themes/LeaveIt/    ───────────►     ├─ posts/xxx/index.html
├─ config.toml        ───────────►     └─ 纯静态 HTML/CSS/JS
└─ public/ ◄──────────Git子模块─────────┘
```

访问地址：https://tifinity.github.io/

### GitHub Pages 工作原理

1. **特殊仓库名**：`Tifinity.github.io` 是 GitHub 的 User Pages 仓库
2. **自动托管**：GitHub 自动为该仓库启用 Pages 功能，从 `master` 分支提供静态文件
3. **CDN 加速**：GitHub 自动将内容分发到全球 CDN

### 自动化部署流程

1. **写作**：在 `Tifinity-Blog/content/` 目录下写 Markdown 文章
2. **推送**：代码推送到 `master` 分支
3. **触发**：Drone CI 检测到推送，自动运行流水线
4. **构建**：使用 `klakegg/hugo` Docker 镜像生成静态站点
5. **部署**：将 `public/` 目录（即子模块）强制推送到 `Tifinity.github.io` 的 `master` 分支
6. **上线**：GitHub Pages 自动更新，博客生效

### 子模块说明

`public/` 目录是 Git 子模块，指向部署仓库：

```bash
# 初始化子模块（首次克隆后）
git submodule update --init --recursive

# public/ 目录实际上就是 Tifinity.github.io 仓库
# 修改 public/ 并提交，等于提交到 Tifinity.github.io
```

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
