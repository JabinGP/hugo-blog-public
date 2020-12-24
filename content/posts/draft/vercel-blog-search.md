---
title: "利用vercel为hugo静态博客添加搜索功能"
date: 2020-12-16T21:04:01+08:00
draft: true
---

## 1. 静态 vs 搜索

静态博客简而言之就是博客中的所有页面都是现成的**html文件**，部署站点时仅仅需要一个**http服务器**即可，这个服务器仅仅用于返回html页面和html中引用的图片等**静态资源**，除此之外服务器并不提供额外的功能如**存储数据**、**数据计算**。

搜索功能的核心有三点：

1. 得到需要搜索的原始数据
2. 对原始数据建立并存储索引
3. 根据搜索请求的关键词，查找已存储的索引

但静态博客的服务器并不具备实现这三点的能力，充其量可以实现**第一点**，这导致了我们的搜索功能并不能依赖静态博客的服务器来完成，需要借助外界的帮助比如：

1. 通过访问博客的**用户本身的电脑环境**计算直接获取结果。
2. 寻求**第三方的服务**，把博客的数据发送给第三方，让第三方帮我们计算并通过网络返回结果。

## 2. 常见的实现方案

### 2.1 利用访客自己的电脑环境计算

`lunr.js`是可以运行在浏览器端的轻量级搜索引擎，类似的还有`fuse.js`，这两个库可以解决以上三点中的**第二点**、**第三点**，而**第一点**可以借助静态博客框架生成的`index.json`完成。

由于我的博客主题自带lunr.js的支持，因此我lunr.js是这两个搜索引擎中我唯一体验过的，在额外添加了一下中文支持库后lunr.js也能够支持中文的搜索，但实测中文搜索效果依然不理想。更让人觉得不舒服的是每次搜索都需要在访客的本地下载、加载索引，当然在index.json不变时，已经下载过的访客可以借助缓存跳过下载这一步，但由于**第二点**中的索引是存储在内存中的，当重新打开页面时，索引就会失效进而导致每次都需要重新执行**第二点**，又由于JavaScript的**单线程特性**，建立索引时会导致网站**卡顿**，并且卡顿时间会随着索引文件增大而延长。

这种实现方式的好处是搜索完全不依赖其他的服务，只要博客能访问搜索就可用，这对于静态博客而言是非常重要的一个优点，但它的中文搜索效果确实比较一般，给人一种有了搜索也好像没啥用的感觉，再加上每次加载索引都会卡顿一下，最终我还是决定弃用了它。

### 2.2 借助第三方服务

第三方服务中比较有名的是[algolia](https://www.algolia.com/)，同时也是引入最简单、搜索效果最好的一种静态博客搜索方式，但algolia并不是慈善家，尽管有免费的额度使用，但免费额度在[官网价位表](https://www.algolia.com/pricing/)上显示的使用额度只有`10UNIT`*每`1UNIT`1000次搜索请求和1000条索引记录容量。虽然对于个人博客而言这个免费使用量应该是绰绰有余了，但不掏钱是不能总是要求有馅饼掉下来吃的，当哪天algolia收费时我将不得不专门为我的搜索支付费用，因此algolia也不是我的理想方案。

## 3. 基于vercel的实现方案

> 这种添加搜索功能的方式可以借助Vercel的Serverless Function部署，也可以部署到其他云函数服务或者自己的VPS上，不限于vercel。但如果有自己的VPS那实现方式就有更好的选择了，这里主要针对希望自己的免费的静态博客能配上一个较好的免费搜索功能的人。

借助vercel的Serverless Functions功能，我们可以自己编写一段能够提供搜索服务的函数来为博客提供搜索功能，但vercel有一些需要注意的问题，详见[有关vercel部署的项目在一段时间空闲后会自动关闭的问题](/posts/code/vercel-stop/)

## 1. 选用工具

- vercel: 部署x内存存储x算力
- golang: riot框架

### 1.1 vercel特性

Vercel向个人提供免费的Serverless Function部署服务。

#### 1.1.1 语言支持

目前支持的语言为：

- Node.js
- Go
- Python
- Ruby

实时信息以官网为准[supported-languages](https://vercel.com/docs/serverless-functions/supported-languages)。

#### 1.1.2 运行环境

函数运行环境最大内存为1GB、最长运行时间为10s，有关函数的运行时可以查看[docs/configuration#project/functions](https://vercel.com/docs/configuration#project/functions)。

#### 1.1.3 注意事项

有关使用的注意事项[fair-use-policy](https://vercel.com/docs/platform/fair-use-policy)，特别注意vercel不允许将服务用于代理、VPN、挖矿。

#### 1.1.4 实际使用上的细节

vercel是没有配套的数据存储功能的，使用vercel的Serverless Function要么通过网络连接其他服务商的数据库，要么放弃数据存储功能。

你可能会想使用sqlite或者json这种简单的存储方式来实现简单的数据存储需求，但在vercel的运行时中，你的程序并不具备写权限，但是通过配置文件引入的文件可以被程序读取到。简而言之，你只能读不能写。

vercel的文档中似乎没有提到的一点是，你的服务并不是一直在线的，当一段时间没有请求时（具体没有测试，大概是十几分钟），vercel会将你的服务关闭，注意是关闭而不是暂时挂起，这意味着你通过内存存储的数据也会被清除还原，等到有新的请求来时，vercel才会重新加载你的服务，这也导致了一段时间不请求后的首次请求会较慢，因为你的程序会被重新加载一遍。

如下的程序使用vercel部署后可以证明，当短时间连续请求多次时，接口返回的是"11"->"111"->"1111"，但一段时间没有请求时再请求，接口返回值会变回"11"。

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

## 4. 基于vercel特点的设计

### 4.1 go语言

1.1.1中讲到了vercel是支持几种语言的，选用go的原因是go有现成的**支持中文**的全文搜索库[riot](https://github.com/go-ego/riot)，可以帮助我们实现搜索功能而无需借助其他外在服务如知名的Elasticsearch，因为我们只需要对**少量**的博客文章数据进行搜索，完全没有必要另外起一个Elasticsearch服务（如果刚好有）为我们的静态博客服务，如果你刚好有这样的服务，那自然是将博客数据导入Elasticsearch最好，本文主要针对的是**无服务器的实现方式**。

## 4. 系统运行逻辑

1. 初始化阶段：发起网络请求，请求指定的index.json所在网络地址
2. 初始化阶段：解析index.json中的数据，建立索引
3. 提供服务阶段：索引初始化完毕，提供搜索服务

### 4.1 初始化

在系统部署到vercel时，系统读取内置的index.json文件，解析为map，通过map中包含的`title`、`content`拼接作为正文后索引，随后索引将被存储在内存中。若服务不重启（参考1.1.4），该索引可以一直提供搜索服务而无需重新加载，当服务在关闭后重启时，以上初始化流程会重新进行。

#### 4.1.1 数据集要求

系统会将地址中的JSON数据视为数组，数组中每个元素为一个文章，系统将该文章解析进`map[string]interface{}`中，这样是为了支持尽可能多的数据类型，系统仅会使用到`title`、`content`作为搜索索引，你可以包含任何的其余数据，系统将原封不动在结果中返回。

所以请注意你的index.json数据中一定要包含的数据项：

- title：文章标题
- content：文章内容

#### 4.1.2 hugo中生成数据集的方法

见另一篇文章[hugo中生成站点数据index.json的方法](https://xxxxxx)。

### 4.2 搜索服务

当系统初始化完成，对外公开的接口就是一个搜索服务，调用地址为`/api/search`，方法为`GET`请求，唯一的必须参数为`keyword`。

#### 4.2.1 使用搜索示例

搜索文章中同时包含`安装`和`Docker`的文章，调用地址为`https://xxxxxxx/api/search?keyword=安装 Docker`，调用方法为`GET`请求。

#### 4.2.2 结果返回

返回结果是一个数组，数组的元素是单个符合搜索条件的数据，该数据会包含index.json中存在的所有数据项。

如一下index.json

```json
[
    {
        "categories": [
            "code"
        ],
        "content": "现象 文章中设置了draft: true，本地hugo server不会看到该文章出现在本地调试的博客中，通过Vercel部署后文章出现在线上的站点中。 原因 有关Vercel对Hugo的处理，可以看看这篇官方文章。 打开Vercel项目的settings，可以看到Build & Development Settings下的hugo指令带有-D参数，这个参数会忽略draft这个标记，导致为编辑完的草稿也被编译、出现在博客中。 解决 选项 值 BUILD COMMAND hugo 开启BUILD COMMAND中的OVERRIDE按钮，手动修改编译命令，由于我的hugo没有其他额外的依赖，直接改成hugo，问题解决。 ",
        "date": "2020.12.18",
        "tags": [
            "vercel",
            "hugo",
            "draft"
        ],
        "title": "解决Vercel部署Hugo时草稿也出现在博客中的问题",
        "uri": "/posts/code/vercel-draft/"
    },
    {
        "categories": [
            "code"
        ],
        "content": "进入容器添加用户 进入容器并且直接运行mongo-shell，选择数据库为admin docker exec -it mongo mongo admin 创建用户并尝试连接 db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},\"readWriteAnyDatabase\"]}); db.auth('admin', '123456') ",
        "date": "2020.12.12",
        "objectID": "/posts/code/docker-install-mongo/",
        "tags": [
            "docker",
            "mongodb"
        ],
        "title": "Docker安装MongoDB",
        "uri": "/posts/code/docker-install-mongo/"
    }
]
```

搜索`hugo 草稿`结果会返回

```json
[
    {
        "categories": [
            "code"
        ],
        "content": "现象 文章中设置了draft: true，本地hugo server不会看到该文章出现在本地调试的博客中，通过Vercel部署后文章出现在线上的站点中。 原因 有关Vercel对Hugo的处理，可以看看这篇官方文章。 打开Vercel项目的settings，可以看到Build & Development Settings下的hugo指令带有-D参数，这个参数会忽略draft这个标记，导致为编辑完的草稿也被编译、出现在博客中。 解决 选项 值 BUILD COMMAND hugo 开启BUILD COMMAND中的OVERRIDE按钮，手动修改编译命令，由于我的hugo没有其他额外的依赖，直接改成hugo，问题解决。 ",
        "date": "2020.12.18",
        "tags": [
            "vercel",
            "hugo",
            "draft"
        ],
        "title": "解决Vercel部署Hugo时草稿也出现在博客中的问题",
        "uri": "/posts/code/vercel-draft/"
    },
]
```

但注意无论你数据中包含什么，系统只会利用`title`和`content`进行搜索。

### 4.3 数据集更新

由于vercel不可写，因此更新index.json中的文件是无法实现的，唯一的更新文件的方式是通过重新部署。当vercel的项目与github的仓库连接时，github仓库更新会触发vercel的自动更新与部署，因此可以借助github的仓库更新来实现vercel项目中的index.json的更新。

每次更新文章都会产生新的index.json，每次手动更新仓库是非常麻烦的，这种非常有规律性的操作则极其适合CI/CD。

#### 4.3.1 Github Actions实现数据更新

当你的文章更新时，会产生新的
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
