---
title: "Vercel Blog Search"
date: 2020-12-16T21:04:01+08:00
draft: true
---

## 1. 选用工具

- vercel: 部署x内存存储x算力
- golang: riot框架

## 2. 系统运行逻辑

系统主要分为三个阶段，分别为：

1. 初始化
2. 提供搜索服务
3. 更新数据集

### 2.1 初始化

在系统部署到vercel时，在环境变量中手动设定`数据集的网络地址`，系统将会通过HTTP请求获取该地址中的JSON数据，解析并建立索引。

|变量名|值|
|-|-|
|SEARCH_DATA_URL|https://xxxxxxxx|

#### 2.1.1 数据集要求

系统会将地址中的JSON数据视为数组，数组中每个元素为一个文章，系统将该文章解析进如下的结构体：

```go
// Article 文章
type Article struct {
    Title      string
    Content    string
    Date       string
    URI        string
    ObjectID   string
    Tags       []string
    Categories []string
}
```

请注意你的数据中一定要包含的数据项为（数据中属性名首字母不需要大写，大写是Golang结构体特性）：

- Title：文章标题
- Content：文章内容
- URI：文章的地址
- ObjectID：文章标识，用做于riot索引数据的标识

#### 2.1.2 hugo中生成数据集的方法

// TODO

### 2.2 搜索服务

当系统初始化完成，对外公开的接口就是一个搜索服务，调用地址为`/api/search`，方法为`GET`请求，唯一的必须参数为`keyword`。

#### 2.2.1 使用搜索示例

搜索文章中同时包含`安装`和`Docker`的文章，调用地址为`https://xxxxxxx/api/search?keyword=安装 Docker`，调用方法为`GET`请求。

#### 2.2.2 结果返回

// TODO

### 2.3 数据集更新

除了初始化时系统会根据`环境变量中设定的数据地址`获取数据并建立索引外，你还可以在系统运行时手动发起更新请求，同样地系统会根据`环境变量中设定的数据地址`判断数据是否发生变化，如果数据已经发生变化则进行数据获取、解析、建立索引，若数据没发生变化则不执行任何操作。

> 如果需要更变数据地址，则需要通过vercel管理后台更换环境变量的值。

这个接口可以让你在数据发生更新时通知系统获取最新数据，通知的方式有很多，比如Github Actions。

#### 2.3.1 Github Actions实现数据更新

由于vercel会向github提供deployment的结果，这会触发Github Actions中的deployment_status事件，因此可以借助这个事件完成数据更新。

```yml
name: Notify search service to update
on: deployment_status
jobs:
    notify:
        env:
            SEARCH_SERVICE_NOTIRY_URL: https://go.jabingp.vercel.app/api/nofity
        if: ${{github.event.deployment_status.state == 'success'}}
        runs-on: ubuntu-latest
        steps:
            - name: Make a request
              run: |
                  curl $SEARCH_SERVICE_NOTIRY_URL

```

> 一开始写if的时候一直出错，在网上查到的都是这么写的`if: github.event.deployment_status.state == "success"`，错误信息为The workflow is not valid. .github/workflows/main.yml (Line: 7, Col: 13): Unexpected symbol: '"success"'. Located at position 41 within expression: github.event.deployment_status.state == "success"，后来发现把"success"双引号换成'success'即可。
