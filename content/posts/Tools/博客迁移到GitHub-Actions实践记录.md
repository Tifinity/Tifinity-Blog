---
title: "博客迁移到 GitHub Actions 实践记录"
date: 2026-03-21T22:30:00+08:00
draft: false
categories: ["Tools"]
tags: ["GitHub Actions", "CI/CD", "Hugo", "部署"]
featured_image:
author: "Tifinity"
---

## 背景

我的博客一直使用 Drone CI 进行自动化部署，但随着 GitHub Actions 的成熟，决定将 CI/CD 流程迁移到 GitHub Actions。整个过程踩了不少坑，记录如下。

## 目标

实现以下工作流：
1. 推送代码到 `master` 分支
2. 自动触发 Hugo 构建
3. 部署到 `Tifinity.github.io` 仓库（GitHub Pages）

## 遇到的坑

### 1. Hugo 版本不兼容

**问题**：使用了 Hugo 0.123.0 版本，但 LeaveIt 主题使用了已弃用的 `.RSSLink` 变量，导致构建失败。

```
ERROR render of "404" failed: ... at <.RSSLink>: can't evaluate field RSSLink in type *hugolib.pageState
```

**解决**：降级 Hugo 到 0.87.0（与原 Drone CI 配置一致）

```yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v3
  with:
    hugo-version: '0.87.0'  # 不要追求最新版
    extended: true
```

### 2. Hugo 命令参数大小写

**问题**：`--baseUrl` 应该是 `--baseURL`（URL 必须大写）

```yaml
# 错误
run: hugo --baseUrl="https://tifinity.github.io/"

# 正确
run: hugo --baseURL="https://tifinity.github.io/"
```

### 3. GITHUB_TOKEN 无法推送到外部仓库

**问题**：使用 `secrets.GITHUB_TOKEN` 部署到外部仓库时报错：

```
The generated GITHUB_TOKEN (github_token) does not support to push to an external repository.
```

**原因**：GITHUB_TOKEN 的作用域仅限于当前仓库，无法推送到其他仓库，即使是你自己的仓库。

**解决**：使用 SSH Deploy Key

### 4. SSH Key 格式问题

**问题**：配置了 deploy key 但仍报错：

```
Error loading key "/home/runner/.ssh/github": error in libcrypto
```

**原因**：复制私钥到 GitHub Secrets 时丢失了换行符，导致格式错误。

**正确的 SSH Key 配置步骤**：

```bash
# 1. 生成密钥
ssh-keygen -t ed25519 -C "blog@github-actions" -f ~/.ssh/blog

# 2. 公钥添加到目标仓库（Tifinity.github.io）
# Settings -> Deploy keys -> Add deploy key
# 勾选 "Allow write access"

# 3. 私钥添加到源仓库 Secrets（Tifinity-Blog）
# Settings -> Secrets and variables -> Actions -> New repository secret
# Name: blog
# Value: cat ~/.ssh/blog 的完整内容（包含换行符）
```

## 最终配置

`.github/workflows/deploy.yml`：

```yaml
name: Deploy Hugo Blog

on:
  push:
    branches:
      - master
  workflow_dispatch:  # 支持手动触发

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.87.0'
          extended: true

      - name: Build
        run: hugo --baseURL="https://tifinity.github.io/"

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        if: github.ref == 'refs/heads/master'
        with:
          deploy_key: ${{ secrets.blog }}
          publish_dir: ./public
          publish_branch: master
          external_repository: Tifinity/Tifinity.github.io
```

## 关键概念理解

### GITHUB_TOKEN vs SSH Key

| 特性 | GITHUB_TOKEN | SSH Deploy Key |
|------|-------------|----------------|
| 来源 | GitHub 自动生成 | 手动生成配置 |
| 作用域 | 当前仓库 | 可配置到任意仓库 |
| 适用场景 | 单仓库操作 | 跨仓库部署 |

### 什么是"外部仓库"

虽然 `Tifinity/Tifinity.github.io` 也是我的仓库，但对 GitHub Actions 来说：
- **当前仓库**：Actions 运行的仓库（Tifinity-Blog）
- **外部仓库**：任何其他仓库（包括 Tifinity.github.io）

GITHUB_TOKEN 被严格限制在当前仓库，这是安全设计。

## 验证部署

合并到 master 后，可以在以下位置查看运行状态：

```bash
# CLI 查看
gh run list --limit 5

# 浏览器查看
open https://github.com/Tifinity/Tifinity-Blog/actions
```

## 总结

1. Hugo 版本要与主题兼容，不要盲目追求最新版
2. 注意命令参数的大小写（--baseURL）
3. 跨仓库部署需要使用 SSH Key，不能使用 GITHUB_TOKEN
4. SSH Key 复制时要保持格式完整（包括换行符）
5. 添加 `workflow_dispatch` 支持手动触发，方便调试
