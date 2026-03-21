# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供在本仓库中工作的指导。

## 项目概述

这是一个使用 [Hugo](https://gohugo.io/) 构建的个人博客，Hugo 是用 Go 编写的静态网站生成器。博客部署在 https://tifinity.github.io/，使用 GitHub Pages 托管。

## 架构

### 目录结构

```
├── archetypes/           # 新文章的内容模板
├── content/              # 博客内容（Markdown 文件）
│   ├── about.md         # 关于页面
│   └── posts/           # 按类别组织的博客文章
├── public/              # 生成的静态站点（Git 子模块）
├── static/              # 静态资源（图片、网站图标）
├── themes/LeaveIt/      # Hugo 主题（Git 子模块）
├── config.toml          # Hugo 配置
└── .drone.yml           # 自动化部署的 CI/CD 流水线
```

### 关键组件

- **Hugo**: 静态站点生成器。使用版本 0.87.0+extended（README 中有说明）
- **LeaveIt 主题**: 简洁、极简风格，集成 Valine 评论系统
- **Valine**: 使用 LeanCloud 后端的评论系统
- **Drone CI**: 自动化构建和部署流水线

### Git 子模块

1. `themes/LeaveIt` - 指向 git@github.com:liuzc/LeaveIt.git
2. `public/` - 指向 git@github.com:Tifinity/Tifinity.github.io.git（GitHub Pages 仓库）

克隆或处理此仓库时，请使用 `--recursive` 或运行 `git submodule update --init --recursive` 来获取子模块。

## 常用命令

### 本地开发

```bash
# 构建站点
hugo

# 使用指定的基础 URL 构建
hugo --baseUrl="http://tifinity.github.io/"

# 启动带实时重载的开发服务器
hugo server

# 创建新内容（使用 archetypes/default.md 模板）
hugo new posts/分类名称/文章标题.md
```

### 子模块管理

```bash
# 克隆后初始化子模块
git submodule update --init --recursive

# 将所有子模块更新到最新版本
git submodule update --remote

# 拉取包含子模块的更改
git pull --recurse-submodules
```

## 部署

博客使用 Drone CI 进行自动部署（配置在 `.drone.yml` 中）：

1. 推送到 `master` 分支时，Drone CI 触发
2. 使用 `klakegg/hugo` Docker 镜像构建站点
3. 将 `public/` 目录强制推送到 `Tifinity/Tifinity.github.io` 仓库部署到 GitHub Pages

部署需要 SSH 密钥作为 Drone 密钥 `SSH_KEY` 存储。

## 内容规范

### 文章前置元数据

文章使用 `archetypes/default.md` 中定义的 YAML 前置元数据：

```yaml
---
title: "文章标题"
date: 2021-01-01T00:00:00+08:00
draft: true              # 设为 false 以发布
categories: ["未分类"]   # 中文分类名称
tags: ["none"]
featured_image:
author: "Tifinity"
---
```

### URL 结构

在 `config.toml` 中通过 `Permalinks` 配置：
- 文章：`/year/filename/`（例如：`/2021/my-post/`）

### 内容组织

文章在 `content/posts/` 下按子目录组织：
- `Linux/`、`Python/`、`Unity3D/`、`AI/` 等

## 配置

`config.toml` 中的关键设置：
- 语言：`zh-CN`（中文）
- 分页：每页 12 篇文章
- 启用 Emoji 支持
- 配置 Valine 评论系统（含应用凭证）
- 菜单项：Blog、Categories、Tags、About
