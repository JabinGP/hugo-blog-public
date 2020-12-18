---
title: "iris的jwt实践，获取jwt、携带jwt、验证jwt、设置过期时间、自定义错误处理函数、格式化错误返回"
date: 2020-12-08T21:21:30+08:00
categories: ["code"]
tags: ["golang", "jwt", "iris", "后端"]
draft: false
---

> 由于jwt原理已经有很多文章提及过了，这里不再赘述，本文主要介绍jwt在iris中的实践，文章的最后会给出完整代码，可以运行起来边测试边看。
> 如果文章对你有帮助，点个赞或者留下评论将会是对我的极大鼓励！

## jwt使用方向

本文将jwt用于登录功能，如果是其他功能需求其实也类似，可以触类旁通。

## iris中jwt的使用思想

iris基于中间件思想设计，对于一些重复性的操作，可以通过注册中间件来完成。对于登录功能，我们自然不希望每个api中手动判断是否过期，是否合法等，因为我们需要一个jwt中间件来完成这些操作。

## 获取jwt中间件

这是一个iris官方提供的jwt中间件：

```go
import "github.com/iris-contrib/middleware/jwt"
```

用法如下：

```go
j := jwt.New(jwt.Config{
    // Extractor属性可以选择从什么地方获取jwt进行验证，默认从http请求的header中的Authorization字段提取，也可指定为请求参数中的某个字段

    // 从请求参数token中提取
    // Extractor: jwt.FromParameter("token"),

    // 从请求头的Authorization字段中提取，这个是默认值
    Extractor: jwt.FromAuthHeader,

    // 设置一个函数返回秘钥，关键在于return []byte("这里设置秘钥")
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
      return []byte("My Secret"), nil
    },

    // 设置一个加密方法
    SigningMethod: jwt.SigningMethodHS256,
  })
```

`jwt.Config`中还有一些可以设置的参数，但是便于理解，以以上三个参数作为演示

## 使用jwt中间件

由于并非所有接口都需要设置登录拦截，所以将中间件用于路由，或者路由组是比较符合业务需求的做法

先给出两个接口：

1. "/getJWT"：生成jwt并返回
2. "/showHello"：没有任何限制，访问就输出"Hello Iris JWT"

```go
package main

import (
  "github.com/kataras/iris/v12"

  "github.com/iris-contrib/middleware/jwt"
)

func main() {
  app := iris.New()

  app.Get("/getJWT", func(ctx iris.Context) {
    // 往jwt中写入了一对值
    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
      "foo": "bar",
    })

    // 签名生成jwt字符串
    tokenString, _ := token.SignedString([]byte("My Secret"))

    // 返回
    ctx.JSON(tokenString)
  })

  app.Get("/showHello", func(ctx iris.Context) {
    ctx.JSON("Hello Iris JWT")
  })

  app.Run(iris.Addr(":8080"))
}

```

访问[http://localhost:8080/getJWT](http://localhost:8080/getJWT)返回一串类似这样的字符串

```json
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw"
```

访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```json
"Hello Iris JWT"
```

现在我们要对`showHello`接口加上jwt验证，需要用到上一小节的jwt中间件

```go
package main

import (
  "github.com/kataras/iris/v12"

  "github.com/iris-contrib/middleware/jwt"
)

func main() {
  app := iris.New()

  j := jwt.New(jwt.Config{
    // Extractor属性可以选择从什么地方获取jwt进行验证，默认从http请求的header中的Authorization字段提取，也可指定为请求参数中的某个字段

    // 从请求参数token中提取
    // Extractor: jwt.FromParameter("token"),

    // 从请求头的Authorization字段中提取，这个是默认值
    Extractor: jwt.FromAuthHeader,

    // 设置一个函数返回秘钥，关键在于return []byte("这里设置秘钥")
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
      return []byte("My Secret"), nil
    },

    // 设置一个加密方法
    SigningMethod: jwt.SigningMethodHS256,
  })

  app.Get("/getJWT", func(ctx iris.Context) {
    // 往jwt中写入了一对值
    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
      "foo": "bar",
    })

    // 使用设置的秘钥，签名生成jwt字符串
    tokenString, _ := token.SignedString([]byte("My Secret"))

    // 返回
    ctx.JSON(tokenString)
  })

  // 注意这里加了j.Serve作为路由的中间件
  app.Get("/showHello", j.Serve, func(ctx iris.Context) {
    ctx.JSON("Hello Iris JWT")
  })

  app.Run(iris.Addr(":8080"))
}
```

访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```cmd
required authorization token not found
```

由于开启了jwt认证，你可能会想到，给`Header`中加入一个字段`Authorization`，然后值为刚刚`getJWT`接口返回的字符串就可以成功通过这个验证了。

### 测试JWT验证

在`Postman`（或者其他测试工具，都一样）中请求的`Headers`中加入一对值

|kEY|VALUE|
|-|-|
|Authorization|eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|

访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```cmd
Authorization header format must be Bearer {token}
```

可能你会纳闷，这个`Authorization header format must be Bearer {token}`是什么意思？按逻辑来讲，我们的做法应该是没有错的。确实，就两次请求的变化来看，`Headers`加入的`Authorization`的的确确被识别到了，但是似乎还有些小问题，根据提示，是格式上还有些出入。

在`Postman`（或者其他测试工具，都一样）中请求的`Headers`中更改`Authorization`值为

|kEY|VALUE|
|-|-|
|Authorization|JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|

这样似乎符合`Bearer {token}`格式了，访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```json
Authorization header format must be Bearer {token}
```

说明格式还是不对。

继续修改`Headers`

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|

访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```json
"Hello Iris JWT"
```

这次终于返回`"Hello Iris JWT"`了，说实话这个格式要求和这个提示有点坑。

### JWT中间件做了什么

可以看到，我们的`/showHello`接口纯粹就是返回`"Hello Iris JWT"`，没有做任何和JWT有关的操作，但是由于我们在路由上使用了JWT中间件，我们的`/showHello`接口具有了`自动JWT验证`的功能，这是JWT中间件在执行我们的接口代码之前，自动帮我们做的。

修改`Headers`中的`Authorization`的值，偷偷把最后一个字符改成其他，看看接口是否能够验证出阿里这是一个假的jwt

|kEY|VALUE|
|-|-|
|Authorization（真）|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|
|Authorization（假）|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uu|

访问[http://localhost:8080/showHello](http://localhost:8080/showHello)返回

```json
signature is invalid
```

可见不合法的jwt是无法通过验证的。

### 获取JWT中的值

JWT中间件在验证失败的时候，接口中的代码不会被执行，请求在JWT验证的时候就被响应`signature is invalid`并结束。
而当JWT中间件验证成功的时候，我们则可以在接口中获取到JWT的信息，我们添加一个接口`/showJWT`，来输出JWT的结构：

```go
app.Get("/showJWT", j.Serve, func(ctx iris.Context) {
  jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
  ctx.JSON(jwtInfo)
})
```

同样需要设置`Headers`带上jwt

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|

访问[http://localhost:8080/showJWT](http://localhost:8080/showJWT)返回

```json
{
    "Raw": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw",
    "Method": {
        "Name": "HS256",
        "Hash": 5
    },
    "Header": {
        "alg": "HS256",
        "typ": "JWT"
    },
    "Claims": {
        "foo": "bar"
    },
    "Signature": "4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw",
    "Valid": true
}
```

可以看到`Claims`里面就有我们签发jwt的时候写入的数据了，如果想要在接口中拿到：

```go
app.Get("/showJWTFoo", j.Serve, func(ctx iris.Context) {
  jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
  foo := jwtInfo.Claims.(jwt.MapClaims)["foo"].(string)
  ctx.JSON(foo)
})
```

设置`Headers`带上jwt

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIifQ.4aQan-XvJBjDUDbCVnuh2P_xy54b2aRKKsgKcHUa8uw|

访问[http://localhost:8080/showJWTFoo](http://localhost:8080/showJWTFoo)返回

```json
"bar"
```

### 设置过期时间

由于jwt签发出去之后，后端并不存储，所以相当于后端并不能管理签发出去的jwt，这种情况下后端自然不能让签发出去的jwt永久有效，需要根据需求设置一个过期时间。

新增一个获取JWT的接口，这次在`Claims`中写入更多的数据：

```go
app.Get("/getJWTWithExp", func(ctx iris.Context) {
  token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
    // 根据需求，可以存一些必要的数据
    "userName": "JabinGP",
    "userId":   "1",
    "admin":    true,

    // 签发人
    "iss": "iris",
    // 签发时间
    "iat": time.Now().Unix(),
    // 设定过期时间，便于测试，设置1分钟过期
    "exp": time.Now().Add(1 * time.Minute * time.Duration(1)).Unix(),
  })

  // 使用设置的秘钥，签名生成jwt字符串
  tokenString, _ := token.SignedString([]byte("My Secret"))

  // 返回
  ctx.JSON(tokenString)
})
```

请求[http://localhost:8080/getJWTWithExp](http://localhost:8080/getJWTWithExp)返回

```json
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwiZXhwIjoxNTc1Mzc5OTk0LCJpYXQiOjE1NzUzNzk5MzQsImlzcyI6ImlyaXMiLCJ1c2VySWQiOiIxIiwidXNlck5hbWUiOiJKYWJpbkdQIn0.AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgg"
```

设置`Headers`带上新的jwt

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwiZXhwIjoxNTc1Mzc5OTk0LCJpYXQiOjE1NzUzNzk5MzQsImlzcyI6ImlyaXMiLCJ1c2VySWQiOiIxIiwidXNlck5hbWUiOiJKYWJpbkdQIn0.AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgg|

访问[localhost:8080/showJWT](localhost:8080/showJWT)返回

```json
{
    "Raw": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwiZXhwIjoxNTc1Mzc5OTk0LCJpYXQiOjE1NzUzNzk5MzQsImlzcyI6ImlyaXMiLCJ1c2VySWQiOiIxIiwidXNlck5hbWUiOiJKYWJpbkdQIn0.AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgg",
    "Method": {
        "Name": "HS256",
        "Hash": 5
    },
    "Header": {
        "alg": "HS256",
        "typ": "JWT"
    },
    "Claims": {
        "admin": true,
        "exp": 1575379994,
        "iat": 1575379934,
        "iss": "iris",
        "userId": "1",
        "userName": "JabinGP"
    },
    "Signature": "AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgg",
    "Valid": true
}
```

隔一分钟后，再次请求[localhost:8080/showJWT](localhost:8080/showJWT)返回

```json
Token is expired
```

说明过期时间设置生效。

### 设置错误返回格式

由于JWT中间件在检验jwt发现不合法时，会自动响应返回错误信息并结束请求，如上所示，错误都是一句话，这样的错误对于前端来说非常不友好，前端更希望能通过一个状态码来判断请求是否顺利。

首先定义一个响应数据的模板：

```go
// ResModel 返回数据模板
type ResModel struct {
  Code string      `json:"code"`
  Msg  string      `json:"msg"`
  Data interface{} `json:"data"`
}
```

获取一个新的中间件，与之前不同的是自定义了错误处理函数（扒了源码改的）:

注意引入的context是iris包下的，否则无法符合错误处理函数的定义。

```go
import "github.com/kataras/iris/v12/context"
```

```go
// j2 对比 j 添加了错误处理函数
j2 := jwt.New(jwt.Config{
  // 注意，新增了一个错误处理函数
  ErrorHandler: func(ctx context.Context, err error) {
    if err == nil {
      return
    }

    ctx.StopExecution()
    ctx.StatusCode(iris.StatusUnauthorized)
    ctx.JSON(ResModel{
      Code: "501",
      Msg:  err.Error(),
    })
  },
  // 设置一个函数返回秘钥，关键在于return []byte("这里设置秘钥")
  ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
    return []byte("My Secret"), nil
  },

  // 设置一个加密方法
  SigningMethod: jwt.SigningMethodHS256,
})
```

在一个新的接口上应用新的`j2`中间件

```go
app.Get("/showJWTErrWithFormat", j2.Serve, func(ctx iris.Context) {
  jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
  ctx.JSON(jwtInfo)
})
```

设置`Headers`带上一个过期的jwt

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwiZXhwIjoxNTc1Mzc5OTk0LCJpYXQiOjE1NzUzNzk5MzQsImlzcyI6ImlyaXMiLCJ1c2VySWQiOiIxIiwidXNlck5hbWUiOiJKYWJpbkdQIn0.AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgg|

访问[localhost:8080/showJWT](localhost:8080/showJWT)返回

```json
{
    "code": "501",
    "msg": "Token is expired",
    "data": null
}
```

篡改`Headers`中`Authorization`的值，把最后一个字符改成其他的：`g`->`o`

|kEY|VALUE|
|-|-|
|Authorization|Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwiZXhwIjoxNTc1Mzc5OTk0LCJpYXQiOjE1NzUzNzk5MzQsImlzcyI6ImlyaXMiLCJ1c2VySWQiOiIxIiwidXNlck5hbWUiOiJKYWJpbkdQIn0.AMFZNbtERLHzKAR9z7oF_0QeNoARKr4dkjocNbVsWgo|

访问[localhost:8080/showJWT](localhost:8080/showJWT)返回

```json
{
    "code": "501",
    "msg": "signature is invalid",
    "data": null
}
```

至此，jwt基本符合使用需求了，最后附上完整的代码。

## 完整代码

> 如果文章对你有帮助，点个赞或者留下评论将会是对我的极大鼓励！

```go
package main

import (
  "time"

  "github.com/kataras/iris/v12"

  "github.com/iris-contrib/middleware/jwt"
  "github.com/kataras/iris/v12/context"
)

// ResModel 返回数据模板
type ResModel struct {
  Code string      `json:"code"`
  Msg  string      `json:"msg"`
  Data interface{} `json:"data"`
}

func main() {
  app := iris.New()

  j := jwt.New(jwt.Config{
    // Extractor属性可以选择从什么地方获取jwt进行验证，默认从http请求的header中的Authorization字段提取，也可指定为请求参数中的某个字段

    // 从请求参数token中提取
    // Extractor: jwt.FromParameter("token"),

    // 从请求头的Authorization字段中提取，这个是默认值
    Extractor: jwt.FromAuthHeader,

    // 设置一个函数返回秘钥，关键在于return []byte("这里设置秘钥")
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
      return []byte("My Secret"), nil
    },

    // 设置一个加密方法
    SigningMethod: jwt.SigningMethodHS256,
  })

  // j2 对比 j 添加了错误处理函数
  j2 := jwt.New(jwt.Config{
    // 注意，新增了一个错误处理函数
    ErrorHandler: func(ctx context.Context, err error) {
      if err == nil {
        return
      }

      ctx.StopExecution()
      ctx.StatusCode(iris.StatusUnauthorized)
      ctx.JSON(ResModel{
        Code: "501",
        Msg:  err.Error(),
      })
    },
    // 设置一个函数返回秘钥，关键在于return []byte("这里设置秘钥")
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
      return []byte("My Secret"), nil
    },

    // 设置一个加密方法
    SigningMethod: jwt.SigningMethodHS256,
  })

  app.Get("/getJWT", func(ctx iris.Context) {
    // 往jwt中写入了一对值
    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
      "foo": "bar",
    })

    // 使用设置的秘钥，签名生成jwt字符串
    tokenString, _ := token.SignedString([]byte("My Secret"))

    // 返回
    ctx.JSON(tokenString)
  })
  app.Get("/getJWTWithExp", func(ctx iris.Context) {
    token := jwt.NewTokenWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
      // 根据需求，可以存一些必要的数据
      "userName": "JabinGP",
      "userId":   "1",
      "admin":    true,

      // 签发人
      "iss": "iris",
      // 签发时间
      "iat": time.Now().Unix(),
      // 设定过期时间，便于测试，设置1分钟过期
      "exp": time.Now().Add(1 * time.Minute * time.Duration(1)).Unix(),
    })

    // 使用设置的秘钥，签名生成jwt字符串
    tokenString, _ := token.SignedString([]byte("My Secret"))

    // 返回
    ctx.JSON(tokenString)
  })

  app.Get("/showHello", j.Serve, func(ctx iris.Context) {
    ctx.JSON("Hello Iris JWT")
  })

  app.Get("/showJWT", j.Serve, func(ctx iris.Context) {
    jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
    ctx.JSON(jwtInfo)
  })

  app.Get("/showJWTFoo", j.Serve, func(ctx iris.Context) {
    jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
    foo := jwtInfo.Claims.(jwt.MapClaims)["foo"].(string)
    ctx.JSON(foo)
  })

  app.Get("/showJWTErrWithFormat", j2.Serve, func(ctx iris.Context) {
    jwtInfo := ctx.Values().Get("jwt").(*jwt.Token)
    ctx.JSON(jwtInfo)
  })

  app.Run(iris.Addr(":8080"))
}
```
