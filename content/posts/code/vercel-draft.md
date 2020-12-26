---
title: "解决Vercel部署Hugo时draft:true不生效，草稿也出现在博客中的问题。"
date: 2020-12-18T09:56:26+08:00
categories: ["code"]
tags: ["vercel", "hugo", "draft"]
draft: false
---

## 现象

文章中设置了`draft: true`，本地`hugo server`不会看到该文章出现在本地调试的博客中，通过Vercel部署后文章出现在线上的站点中。

## 原因

> 有关Vercel对Hugo的处理，可以看看[这篇官方文章](https://vercel.com/guides/deploying-hugo-with-vercel)。

打开Vercel项目的settings，可以看到`Build & Development Settings`下的hugo指令带有`-D`参数，这个参数会忽略draft这个标记，导致为编辑完的草稿也被编译、出现在博客中。

## 解决

|选项|值|
|-|-|
|BUILD COMMAND|hugo|

开启`BUILD COMMAND`中的`OVERRIDE`按钮，手动修改编译命令，由于我的hugo没有其他额外的依赖，直接改成`hugo`，问题解决。
