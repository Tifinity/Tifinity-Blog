---
title: "Git快速设置"
date: 2019-12-24T14:12:21+08:00
draft: false
categories: ["工欲善其事，必先利其器"]
tags: ["Git"]
featured_image: https://github.com/Tifinity/MyImage/raw/master/Git/logo.jpg
author: "Tifinity"
---

# Git的一些让人方便的设置

~~~shell
# 用户名和邮箱
git config --global user.name <your name>
git config --global user.email <your_email@example.com>
# 解决中文在Git Bash里显示诸如“/202/233/233”的转码问题
# core.quotepath 为false的话，就不会对0x80以上的字符进行quote。中文显示正常。
git config --global core.quotepath false
# 设置commit的默认编辑器
git config --global core.editor /usr/bin/vim
# 避免每次pull和push时输入用户名和密码
git config --global credential.helper store
# Windows Git Bash启用http/https协议
git config --global credential.helper wincred
# 设置大小写敏感，保持Mac/Win/Linux一致性
git config --global core.ignorecase false
# Windows CRLF
git config --global core.autocrlf true
# Linux，Mac
git config --global core.autocrlf input
~~~

