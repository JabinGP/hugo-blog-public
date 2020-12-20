---
title: "有关vercel部署的项目在一段时间空闲后会自动关闭的问题"
date: 2020-12-19T14:41:49+08:00
draft: false
categories: ["code"]
tags: ["serverless functions","serverless", "vercel", "stateless"]
---

## 现象

通过vercel部署了Go的Serverless Functions，其中init函数中包含较长时间的初始化过程，每当一段时间不访问服务后的第一次访问总会很慢，猜测vercel是会将一段时间没人使用的服务关闭，等待该服务再次被请求时重新开启。

## 验证

如下的程序使用vercel部署后可以证明，当短时间连续请求多次时，接口返回的是
"11"->"111"->"1111"，但一段时间没有请求时再请求，接口返回值会变回"11"。

```go
import (
    "fmt"
    "net/http"
)

var count = "1"

func Handler(w http.ResponseWriter, r *http.Request) {
    count += "1"
    fmt.Fprintf(w, count)
}
```

## 结论

云函数对比传统开发模式而言多了一个"装载"的过程，即请求到达时，云函数先被装载到一个临时环境中，并在该环境中运行，为提高资源利用率，该云函数会继续占用该临时环境一段时间，以因对随后可能的连续请求（类似缓存原理），尽管此时云函数因为占有内存而在内部拥有保存运行状态的能力，但随着云函数空闲，云函数被移除该临时环境，对应的内存也会被清除回收，原先程序的运行数据将被不被保存，因此在编写云函数时应该保证该函数是无状态的，即每次请求之间应该不相互依赖，否则云函数将无法保证运行的结果能够符合预期。

> 发现一篇有关这个问题的文章，写的很详尽，推荐：[万物皆可 Serverless 之关于云函数冷热启动那些事儿](https://blog.csdn.net/weixin_42409476/article/details/106789951)
