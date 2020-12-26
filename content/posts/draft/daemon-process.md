---
title: "操作系统系列之——守护进程（Daemon Process）"
date: 2020-12-22T21:24:06+08:00
tags: ["os", "procedure", "process", "unix-like", "c"]
categories: ["code", "os"]
draft: true
---

> 本文要求你对进程有一定概念。如果不懂什么是进程，推荐你查看我的上一篇文章[操作系统系列之——程序和进程](https://blog.jabingp.cn/posts/code/linux-process/)。

## 1. 概念解释

**守护进程**的英文原文为*Daemon Process*，在类Unix系统世界里代表着**运行在后台**不懈地处理系统琐事的进程。

## 1.1. Daemon一词的起源

**守护进程**的英文原文为*Daemon Process*，其中*Daemon*源自古典拉丁语，表示**具有掌管能力的灵体**。

该词曾被科学家**詹姆斯·麦克斯韦**引用于一个“思想实验”中，他想象在一个被分隔为两部分的封闭容器中间，存在一个大小仅仅够一个气体分子通过的门。这道门，由一个想象中的掌管精灵Daemon控制。这个掌管精灵Daemon根据气体的速度，只让速度快(温度高)的气体分子由A半到B半，也只让速度慢(温度低)的气体由B半到A半。最终这个封闭容器将一半冷一半热。麦克斯韦的这个想法因为脱离了热力学原理而轰动一时，但最终量子力学理论证明了其不可能性。

随后在1963年运行于`IBM 7094`系统上的`Project MAC`项目中，具有物理学背景的Jerome H. Saltzer教授用麦克斯韦实验中那个不断看管着分子的掌管精灵，来指代**不懈地工作**来执行系统琐事的**后台进程**，这是*Daemon*一词首次出现在计算机世界。

> 有关*Daemon*的解释来源于[Daemon 起源：中文翻译](https://linuxtoy.org/archives/the-origin-of-the-word-daemon.html)
> 以及[Daemon 起源：原文](http://ei.cs.vt.edu/~history/Daemon.html)。

## 1.2. **守护进程**等于**后台进程**吗

后台进程不等于守护进程。


> [后台进程不等于守护进程](https://www.cnblogs.com/SophiaTang/archive/2011/11/25/2263654.html)

## 2. 启动一个守护进程
