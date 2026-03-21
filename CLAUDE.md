# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog built with [Hugo](https://gohugo.io/), a static site generator written in Go. The blog is published at https://tifinity.github.io/ using GitHub Pages.

## Architecture

### Directory Structure

```
├── archetypes/           # Content templates for new posts
├── content/              # Blog content (markdown files)
│   ├── about.md         # About page
│   └── posts/           # Blog posts organized by category
├── public/              # Generated static site (git submodule)
├── static/              # Static assets (images, favicons)
├── themes/LeaveIt/      # Hugo theme (git submodule)
├── config.toml          # Hugo configuration
└── .drone.yml           # CI/CD pipeline for automated deployment
```

### Key Components

- **Hugo**: Static site generator. Uses version 0.87.0+extended (noted in README)
- **LeaveIt Theme**: Clean, minimal theme with Valine comment integration
- **Valine**: Comment system using LeanCloud backend
- **Drone CI**: Automated build and deployment pipeline

### Git Submodules

1. `themes/LeaveIt` - Points to git@github.com:liuzc/LeaveIt.git
2. `public/` - Points to git@github.com:Tifinity/Tifinity.github.io.git (the GitHub Pages repo)

When cloning or working with this repo, use `--recursive` or run `git submodule update --init --recursive` to fetch submodules.

## Common Commands

### Local Development

```bash
# Build the site
hugo

# Build with specific base URL
hugo --baseUrl="http://tifinity.github.io/"

# Start development server with live reload
hugo server

# Create new content (uses archetypes/default.md template)
hugo new posts/category-name/post-title.md
```

### Submodule Management

```bash
# Initialize submodules after clone
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote

# Pull changes including submodules
git pull --recurse-submodules
```

## Deployment

The blog uses Drone CI for automated deployment (configured in `.drone.yml`):

1. On push to `master` branch, Drone CI triggers
2. Builds the site using `klakegg/hugo` Docker image
3. Deploys to GitHub Pages by force-pushing `public/` directory to `Tifinity/Tifinity.github.io` repo

The deployment requires an SSH key stored as Drone secret `SSH_KEY`.

## Content Guidelines

### Post Frontmatter

Posts use YAML frontmatter defined in `archetypes/default.md`:

```yaml
---
title: "Post Title"
date: 2021-01-01T00:00:00+08:00
draft: true              # Set to false to publish
categories: ["未分类"]   # Chinese category names
tags: ["none"]
featured_image:
author: "Tifinity"
---
```

### URL Structure

Configured in `config.toml` with `Permalinks`:
- Posts: `/year/filename/` (e.g., `/2021/my-post/`)

### Content Organization

Posts are organized in subdirectories under `content/posts/`:
- `Linux/`, `Python/`, `Unity3D/`, `AI/`, etc.

## Configuration

Key settings in `config.toml`:
- Language: `zh-CN` (Chinese)
- Pagination: 12 posts per page
- Emoji support enabled
- Valine comment system configured with app credentials
- Menu items: Blog, Categories, Tags, About
