---
title: "Windows下使用startup.bat启动Tomcat输出乱码的原因探究、解决方案"
date: 2020-12-08T21:25:28+08:00
images: []
categories: ["code"]
tags: ["java", "tomcat", "windows", "乱码"]
draft: false
---

![startup.bat输出乱码](1.png)

首先找到输出日志的配置文件
![输出日志配置文件】](2.png)

打开`logging.properties`，搜索`log`，可以发现以下配置
![配置文件](3.png)

再看看启动bat的cmd的属性
![cmd属性](4.png)

很明显编码是GBK，所以乱码的原因实锤了，知道了原因就好解决了

1. 把`cmd`的编码改为`utf-8`
2. 把`tomcat`的日志输出改成`GBK`

由于方法`2`改配置文件比较方便，并且对其他程序没有影响，故选用`2`

修改`logging.properties`

```properties
java.util.logging.ConsoleHandler.encoding = UTF-8
```

改为

```properties
java.util.logging.ConsoleHandler.encoding = GBK
```

保存文件，重启bat

![正常输出](5.png)

不再乱码了，大功告成！
