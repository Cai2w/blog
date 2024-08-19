---
title: hexo-butterfly挂载自己的网页？
date: 2021-11-03 17:08:05
tags:
- hexo
categories:
- 小知识
description:
sticky:
cover: https://cdn.jsdmirror.com/gh/Cai2w/cdn/img/undraw_Code_thinking_re_gka2.png
---

> 如果你想把自己写的静态网页也放到博客中，那这篇文章或许能帮助你

步骤：

1. 前往你的 Hexo 博客的根目录

2. 输入`hexo new page tags`

3. 你会找到`source/tags/index.md`这个文件
4. 删除这个文件，只保留文件夹
5. 将写好的网页内容放到该文件夹中
6. 在`_config.butterfly.xml`的`menu`中加上你创建的文件夹名，以及喜欢的图标
7. 在`_config.xml`的`skip_render`选项中添加 `- “tags/**”`

