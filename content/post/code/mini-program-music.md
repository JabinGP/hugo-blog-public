---
title: "在微信小程序使用播放音乐的方法，以及微信小程序播放背景音乐失败的解决方案汇总"
date: 2020-12-08T20:08:59+08:00
images: []
categories: ["code"]
tags: ["javascript", "小程序", "网易云音乐", "音乐外链"]
authors: []
---

>项目要做一个可以为日记添加音乐的小程序，所以要用到音乐api，参考了一些文章后我们封装了一个qq音乐api库（完成了动态token获取，音乐搜索，音乐专辑图片，音乐名称，歌手名称，播放），有需要的可以到Github自提。  

小程序qq音乐api库Gihub地址[https://github.com/FisherWY/QQMusicPlugin](https://github.com/FisherWY/QQMusicPlugin)，里面有简单的教程，如果开发工具不勾选ES6转ES5的话，则可以不需要里面的es6-promise这个js文件，并把`var Promise = require('./es6-promise.min.js')`在qqMusicTools.js中去掉。

### 播放背景音乐失败的解决方案  

1. 没有为音乐设置title
    解决方案：在设置背景音乐的时候设置title:"随便设置点东西"

2. 请求的url中带有中文路径
    使用encodeURI("xxxxxx")转码

3. 手机设置了静音模式

4. 一个非常奇葩的问题（翻遍了互联网都没找到解决方案，怀疑是官方的bug了）
    android端（移动数据、WiFi、热点一切正常），电脑模拟器（开WiFi，3G各种模式都正常）都可以正常播放，iOS使用WiFi时正常播放，iOS使用移动数据、热点的时候无法播放，报错如下：

    ```javascript
    errCode:10002
    errMsg:"playerErrCode:6, systemErrCode:403, domain:com.tencent.KSAudioPlayer.HTTP, description:未能完成操作。（“com.tencent.KSAudioPlayer.HTTP”错误 403。）"
    src:"http://ws.stream.qqmusic.qq.com/C400002WqezQ4dmIeT.m4a?guid=126548448&vkey=0E12BA0C521F05EF0103E99180DC5C50CA0E942E3183546F5D186F3E6F20F161E9EB0DCEA038F0A9A578E2DFAEBF434AF48521DA440A7EFF&fromtag=0"
    ```

    ~~暂时没有找到解决方案，但是问题只在qq音乐api上出现，使用网易云完全正常。下一步准备使用网易云api代替qq音乐api~~。已经开发新的网易云api代替qq音乐api了，需要的可以看看这个项目：[为微信小程序开发的网易云音乐api库](https://github.com/JabinGP/NetEaseCloudMusicApi)。
