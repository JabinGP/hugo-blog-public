---
title: "解决VSCode开发Golang时无法保存的问题"
date: 2020-12-18T20:54:11+08:00
draft: false
tags: ["vscode", "golang", "gopls", "proxy", "router"]
categories: ["code"]
---

## 运行环境

- 操作系统：macOS 10.15.6
- Golang版本：go version go1.14.5 darwin/amd64
- VSCode版本：1.52.1

## 问题描述

开发时启用gomodule、开启了gopls，在保存代码时频繁出现：

> Saving 'main.go': Getting code actions from ''Go'' (configure)

这不仅仅是没有代码提示、没有代码格式化的问题，连保存都无法生效，搜索了Google发现了在Github issues中有挺多关于这个问题的讨论和可以尝试的解决方案，这里给出几篇比较有参考价值的：

- [https://github.com/microsoft/vscode-go/issues/3105](https://github.com/microsoft/vscode-go/issues/3105)
- [https://github.com/microsoft/vscode-go/issues/3179](https://github.com/microsoft/vscode-go/issues/3179)
- [https://github.com/golang/vscode-go/issues/236](https://github.com/golang/vscode-go/issues/236)

## 问题解决

很遗憾的是上面的链接中的方法我都有尝试，但是都没有效果，直到我看到了一条[回复](https://github.com/golang/vscode-go/issues/236#issuecomment-665753854)，原文如下：

> I have a new discovery.
> I also have the problem of Frequent pop-up on save: "Getting code actions: Go, Go, ...", I'm Chinese, so in our country,google is outside our great fire wall... I can't open google.com unless I use a proxy.
> Today I opened vscode, the problem "Getting code actions: Go, Go, ..." came out as usual. I closed vscode and reopen it, the problem still there... After a few times reopening vscode, suddenly I find out that I'm not using proxy in vscode!!! So I turn on proxy, and, the miracle happen--the problem had gone, I can use gopls server fastly!!! All the things be OK!
> Then After a few minutes, I closed vscode, and waited for some minutes. I reopened it, everything still working fine!!!

简单概括一下就是他到了一个不需要科学上网的地方后，问题就消失了。

我看到这条issues后，只是猜测当保存时会触发某种请求，请求的服务器可能无法在国内直连，并且有了一次请求成功后就不会再触发该请求了。但是因为我日常都挂着科学上网，理应不存在代理导致的问题，但是我又突然想起来一个问题，该请求是保存时触发的，而我的代理通常默认只对浏览器等会自动走代理的软件有效，特别是命令行终端就不会自动走代理（详见[如何让命令行终端的网络请求也走代理（未上线）]()），我突然想到其实VSCode里面自动触发的很多操作并不会走代理，我依然可能存在代理的问题。

想到这里我就给我的路由器挂上了代理，因为路由器会将所有数据包做代理，但凡是网络请求就逃不过，这样就可以保证VSCode自动触发的那些命令也能被代理了。

> A new version of gopls (v0.6.1) is available. Please update for an improved experience.

结果是在我挂完代理后再打开VSCode编辑Golang代码、保存，我得到了一个gopls的更新提示，问我需不需要更新新版gopls，我选择了更新后一切使用正常，关掉路由器代理后一切也依旧正常，问题解决。
