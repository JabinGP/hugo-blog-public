---
title: "Go语言ORM框架快速上手，ORM操作Mysql数据库示例"
date: 2020-12-08T21:38:19+08:00
images: []
categories: ["code"]
tags: ["mysql", "orm", "gorm", "xorm", "database"]
authors: []
---

> 直接上Github[https://github.com/JabinGP/demo-gomysql](https://github.com/JabinGP/demo-gomysql)

## 说明

代码有master和gorm两个分支，master分支用的是xorm，gorm不言而喻。

两个分支都是简单的单表查询，比较便于理解学习框架。

两个分支都是只需要补齐mysql的配置文件，提前建好对应库，不需要建表就可以直接跑起来的，便于快速看到效果，具体的启动方式在README中有解释。

## 使用感受

> 具体的就不多说了，都在代码里面，也有详细的注释，主要讲讲体验。
> 如果你没有决定用哪个，信我就选`xorm` 。

### 单表

两个框架大同小异，但是个人感觉xorm的文档，以及api的逻辑性上比较顺应我的想法（大概就是我猜它会怎么去暴露api给我用，然后用了之后发现确实是这样的）。

Gorm有几点我觉得不太好用：

1. 为了实现软删除，Gorm自带一个model，官方推荐组合使用struct，但是实际上使用了这个以后，结构体字面量就无法直接指定id了。。{ID:xxx}会报语法错误，如果要嵌套使用{{ID:xx}}就会爆参数不完整的错误，这样在使用结构体指定字段查询的时候不能直接指定id就非常麻烦，必须先初始化一个结构体，在xxx.ID=xxx指定。
2. select找不到记录的时候竟然返回了err，这个我觉得不太合理，找不到值怎么能是错误呢。

### 多表

#### Gorm

Gorm在我的一个简易demo[iris项目实战简易聊天室](https://github.com/JabinGP/demo-chatroom)中使用到，
其中Gorm在另一个简单的demo中尝试使用，Gorm在项目中用到级联查询的时候非常难受，翻看了很多遍官方文档，官方文档在这一块说得不是很清楚，也没有什么示例，最后是自己不断尝试组合实现的。

#### Xorm

由于还没有使用过Xorm做项目，只是写了简单的单表示例，Xorm似乎也支持级联查询，但我更看重的是Xorm在文档中提供了直接执行SQL的方法，我更倾向于使用原生SQL加传入结构体指针扫描的形式完成级联查询，接下来打算将[iris项目实战简易聊天室](https://github.com/JabinGP/demo-chatroom)用xorm重构，看看效果后再对本文进行更新，不过个人感觉xorm应该是会更好用的。
