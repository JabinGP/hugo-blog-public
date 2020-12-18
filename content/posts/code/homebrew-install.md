---
title: "使用homebrew安装你电脑上的所有软件吧"
date: 2020-12-08T21:42:54+08:00
categories: ["code"]
tags: ["homebrew", "macbook", "macos"]
draft: false
---

最近需要重装 mac，想着能不能用 homebrew 统一安装所有软件，本来以为有些软件应该会缺失的，结果竟然几乎完全可以用 brew install，简单记录供日后参考。

## 已尝试可安装软件

atom、datagrip、deno、discord、docker、emacs、golang、chrome、idea、网易邮箱大师（mailmaster）、网易云音乐、有道词典、micro、mos、mumble、nginx、node、openjdk8、openjdk11、postman、qq、qqmusic、rust-up、sogouinput、ssr、telegram、telnet、腾讯会议（tencent-meeting）、terminus、tree、unrar、vscode、wechat、wget、aliwangwang

安装命令可以拉倒最底下查看

## 已尝试失败软件

随着尝试软件的增加，已经出现一些不能使用brew安装的软件了

- 百度网盘，brew安装的是一个很老的叫百度同步盘的东西
- wps，brew安装的是英文版，难用
- zoom会议，brew没有

## 安装 homebrew

访问[homebrew 官网](https://brew.sh/)，复制脚本执行命令即可，这里给出命令，不过还是以官网为准。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

## 用 homebrew 安装所有软件

### Chrome

不多解释了，程序员必备浏览器

```bash
brew cask install google-chrome
```

### 音乐播放器

网易云音乐

```bash
brew cask install neteasemusic
```

QQ 音乐

```bash
brew cask install qqmusic
```

### 通讯工具

微信

```bash
brew cask install wechat
```

QQ

```bash
brew cask install qq
```

mumble，这是一个开源的游戏语音工具，可以在自己的云服务器部署。

```bash
brew cask install mumble
```

### 终端相关

terminus，一个很漂亮的终端。

```bash
brew cask install terminus
```

oh-my-zsh，最好访问[官网](https://ohmyz.sh/)获取最新安装命令。

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 网络相关

测试 socket 代理必备客户端。

```bash
brew cask install shadowsocksx-ng-r
```

Nginx，高性能 Web 服务器。

```bash
brew install nginx
// 通过brew安装的nginx可以通过brew services管理
brew services start nginx
```

### 编程应用

VSCode

```bash
brew cask install visual-studio-code
```

Postman

```bash
brew cask install postman
```

IDEA

```bash
brew cask install intellij-idea
```

### 开发环境

Python

```bash
macOS自带python，无需安装。
```

Java，因为最新版 JDK8 需要付费，这里选择 AdoptOpenJDK。

```bash
brew cask install adoptopenjdk8
```

Golang

```bash
brew install golang
```

Node.js

```bash
brew install node
```

Deno

```bash
brew install deno
```

Rust，根据官网的教程，需要先安装 rustup，再通过 rustup 安装 rust 工具链。

```bash
brew install rustup-init
// 安装完通过rustup-init命令执行安装脚本
rustup-init
```

Docker，有了 Docker 就不用装 Mysql、Redis、MongoDB、Kafka、ZooKeeper 等服务了，直接起 Docker 容器。

```bash
brew cask install docker
```

## 安装命令集合

```bash
brew cask install atom
brew cask install datagrip
brew install deno
brew cask install discord
brew cask install docker
brew install emacs
brew install golang
brew cask install google-chrome
brew cask install intellij-idea
brew cask install mailmaster
brew install micro
brew cask install mos
brew cask install mumble
brew cask install neteasemusic
brew install nginx
brew services start nginx
brew install node
brew cask install adoptopenjdk8
brew install openjdk@11
brew cask install popo
brew cask install postman
brew cask install QQ
brew cask install qqmusic
brew install rustup-init
brew cask install sogouinput
brew cask install shadowsocksx-ng-r
brew cask install telegram
brew install telnet
brew cask install tencent-meeting
brew cask install terminus
brew install tree
brew install unrar
brew cask install visual-studio-code
brew cask install wechat
brew install wget
brew cask install youdaodict
brew cask install aliwangwang
```
