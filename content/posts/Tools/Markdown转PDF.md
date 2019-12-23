---
title: "Markdown转PDF"
date: 2019-12-21T17:39:48+08:00
draft: false
categories: ["工欲善其事，必先利其器"]
tags: ["Markdown", "PDF"]
author: "Tifinity"
---

# Markdown转PDF

### 使用Pandoc

Pandoc被誉为“瑞士军刀”，可以实现很多文本格式的转换。

建议使用方法：先转docx，再用WPS或者Word输出为PDF，只需要Pandoc，不需要任何其他配置。

~~~shell
pandoc filename.md -o filename.docx
~~~

[Pandoc官网](https://www.pandoc.org/)，不过下载速度很慢

可以使用Python的pip来安装：

~~~shell
pip install pandoc
~~~

然后可以在Python中使用Pandoc。

### 使用VSCode + MarkdownPDF插件

在VSCode的扩展中搜索 MarkdownPDF 下载即可。

使用方法为在VSCode中打开.md文件，右键，可以输出为PDF还有一些其它格式。

