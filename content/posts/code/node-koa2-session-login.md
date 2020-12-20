---
title: "Node.js的Koa2中如何使用Session完成登录功能？"
date: 2020-12-08T21:08:05+08:00
categories: ["code"]
tags: ["login","session","koa","koa2","JavaScript","cookie"]
draft: false
---

> 项目要用到登录注册，就需要使用到Cookie和Session来保持登录状态，于是就简单研究了一下

### Cookie和Session的工作原理

前面已经专门发过一篇帖子记录Cookie和Session的工作原理了，不明白的小伙伴可以看看[Cookie、Session是如何保持登录状态的？](https://segmentfault.com/a/1190000019065025)。

### 使用Koa的Session中间件

Koa是一个简洁的框架，把许多小功能都拆分成了中间件，用一个洋葱模型保证了中间件丰富的可拓展性，我们要使用Session来保持登录状态，就需要引用Session中间件。

### 安装Koa-Session中间件

```cmd
npm install koa-session --save
```

如果需要使用TypeScript进行开发，则需要引入对应的TS类型声明

```cmd
npm install @types/koa-session --save
```

### 配置Session

Koa-Session需要做一些配置：

```JavaScript
const session_signed_key = ["some secret hurr"];  // 这个是配合signed属性的签名key
const session_config = {
    key: 'koa:sess', /**  cookie的key。 (默认是 koa:sess) */
    maxAge: 4000,   /**  session 过期时间，以毫秒ms为单位计算 。*/
    autoCommit: true, /** 自动提交到响应头。(默认是 true) */
    overwrite: true, /** 是否允许重写 。(默认是 true) */
    httpOnly: true, /** 是否设置HttpOnly，如果在Cookie中设置了"HttpOnly"属性，那么通过程序(JS脚本、Applet等)将无法读取到Cookie信息，这样能有效的防止XSS攻击。  (默认 true) */
    signed: true, /** 是否签名。(默认是 true) */
    rolling: true, /** 是否每次响应时刷新Session的有效期。(默认是 false) */
    renew: false, /** 是否在Session快过期时刷新Session的有效期。(默认是 false) */
};
```

我们需要关注这几个配置：

- renew rolling
这两个都可以在用户访问的过程中刷新有效期，不至于让用户访问过程中Session过期成为未登录状态
- signed
这个是对客户端Cookie的签名，也就是用一个特点的字符加密，保证客户端Cookie不会被伪造出来
- httpOnly
打开这个使得通过程序(JS脚本、Applet等)无法读取Cookie，大大提高了安全性
- maxAge
以ms为单位的过期时间

### 简单的使用

首先理一下思路

1. 判断访问者的Session有没有过登录记录属性
2. 如果有且值为true，则为已登录，否则为未登录
3. 如果为已登录，则不执行判断，直接返回已登录，如果为未登录，则执行下一步登录验证
4. 如果验证成功，则返回的登录成功，并且在它的session中记下登录属性为true，如果验证失败，则返回登录失败。

> 为了测试方便，以下用Get请求和一个固定的账号密码代替数据库查询，实际开发应该使用POST和数据库比对。同时为了测试方便，将过期时间设置为4000ms，便于快速看到Cookies过期，实际开发应该设置长一些，比如几小时甚至几天，取决于业务需求。

```JavaScript
const Koa = require('koa');                               // 导入Koa
const Koa_Session = require('koa-session');   // 导入koa-session     
// 配置
const session_signed_key = ["some secret hurr"];  // 这个是配合signed属性的签名key
const session_config = {
    key: 'koa:sess', /**  cookie的key。 (默认是 koa:sess) */
    maxAge: 4000,   /**  session 过期时间，以毫秒ms为单位计算 。*/
    autoCommit: true, /** 自动提交到响应头。(默认是 true) */
    overwrite: true, /** 是否允许重写 。(默认是 true) */
    httpOnly: true, /** 是否设置HttpOnly，如果在Cookie中设置了"HttpOnly"属性，那么通过程序(JS脚本、Applet等)将无法读取到Cookie信息，这样能有效的防止XSS攻击。  (默认 true) */
    signed: true, /** 是否签名。(默认是 true) */
    rolling: true, /** 是否每次响应时刷新Session的有效期。(默认是 false) */
    renew: false, /** 是否在Session快过期时刷新Session的有效期。(默认是 false) */
};

// 实例化
const app = new Koa();
const session = Koa_Session(session_config, app)
app.keys = session_signed_key;

// 使用中间件，注意有先后顺序
app.use(session);

app.use(ctx => {
    const databaseUserName = "testSession";
    const databaseUserPasswd = "noDatabaseTest";
    // 对/favicon.ico网站图标请求忽略
    if (ctx.path === '/favicon.ico') return;

    if (!ctx.session.logged) {  // 如果登录属性为undefined或者false，对应未登录和登录失败
        // 设置登录属性为false
        ctx.session.logged = false;

        // 取请求url解析后的参数对象，方便比对
        // 如?nickname=post修改&passwd=123解析为{nickname:"post修改",passwd:"123"}
        let query = ctx.request.query;

        // 判断用户名密码是否为空
        if (query.nickname && query.passwd) {

            // 比对并分情况返回结果  
            if (databaseUserName == query.nickname) {  // 如果存在该用户名

                // 进行密码比对并返回结果 
                ctx.body = (databaseUserPasswd == query.passwd) ? "登录成功" : "用户名或密码错误";
                ctx.session.logged = true;
            } else {                    // 如果不存在该用户名                                           //  如果用户名不存在
                ctx.body = "用户名不存在";
            }
        } else {
            ctx.body = "用户名密码不能为空";
        }
    } else {
        ctx.body = "已登录";
    }

}
);

app.listen(3000);
console.log("Koa运行在：http://127.0.0.1:3000");   
```

运行一下，控制台输出：

```cmd
Koa运行在：http://127.0.0.1:3000
```

访问[http://127.0.0.1:3000](http://127.0.0.1:3000)，可以看到我们没有填写登录参数，然后返回了用户名密码不能空，并且按下F12，点击Cookies再点击[http://127.0.0.1:3000](http://127.0.0.1:3000)，看到了我们的SessionId被记录到了Cookies中，说明Session生效了。
![1](https://upload-images.jianshu.io/upload_images/14225973-8ae8fb28dcfe78a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们静置一会但不刷新页面，再点击Cookies后重新点击，[http://127.0.0.1:3000](http://127.0.0.1:3000)（刷新Cookies显示），发现我们的SessionId不见了，说明我们的过期时间也生效了
![2](https://upload-images.jianshu.io/upload_images/14225973-ecbf3753b37436c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我将Cookies的数据截详细一点就是这样的，可以看到有个过期时间：
![3](https://upload-images.jianshu.io/upload_images/14225973-422c47b819d5c7cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们再访问[http://127.0.0.1:3000/?nickname=123](http://127.0.0.1:3000/?nickname=123)和[http://127.0.0.1:3000/?passwd=123](http://127.0.0.1:3000/?passwd=123)，都输出了
![4](https://upload-images.jianshu.io/upload_images/14225973-618d9f63eaf7d2cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

访问[http://127.0.0.1:3000/?nickname=123&&passwd=123](http://127.0.0.1:3000/?nickname=123&&passwd=123)，输出
![5](https://upload-images.jianshu.io/upload_images/14225973-04fc4fc688e30868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

访问[http://127.0.0.1:3000/?nickname=testSession&&passwd=123](http://127.0.0.1:3000/?nickname=testSession&&passwd=123)，输出
![6](https://upload-images.jianshu.io/upload_images/14225973-236ee36413c040fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后尝试正确的用户名密码[http://127.0.0.1:3000/?nickname=testSession&&passwd=noDatabaseTest](http://127.0.0.1:3000/?nickname=testSession&&passwd=noDatabaseTest)
，输出![登录成功](https://upload-images.jianshu.io/upload_images/14225973-a67d3a9bdbc9b3ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
尝试再次访问[http://127.0.0.1:3000/?nickname=testSession&&passwd=noDatabaseTest](http://127.0.0.1:3000/?nickname=testSession&&passwd=noDatabaseTest)
，输出

![重复登录](https://upload-images.jianshu.io/upload_images/14225973-3fbd253861481a36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在有效期限内访问别的页面[http://127.0.0.1:3000/](http://127.0.0.1:3000/)，输出
![有效期内](https://upload-images.jianshu.io/upload_images/14225973-8783aac616ff79aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有效期内不断刷新就能保持登录状态
![不断刷新](https://upload-images.jianshu.io/upload_images/14225973-5152a1573021dfe8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有效期内没有重新操作页面刷新状态就会自然过期
![过期](https://upload-images.jianshu.io/upload_images/14225973-9853d8e5e14bf4df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
