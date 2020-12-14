---
title: "Ubuntu使用apt安装最新版golang"
date: 2020-12-09T21:42:29+08:00
images: []
categories: ["code"]
tags: ["apt", "ubuntu", "golang"]
draft: false
---


### 1. 获取最新的软件包源，并添加至当前的apt库

```cmd
add-apt-repository ppa:longsleep/golang-backports
```

### 2. 更新apt库

```cmd
apt-get update
```

### 3. 安装go

```cmd
sudo apt-get install golang-go
```
