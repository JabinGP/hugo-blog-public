---
title: "记一次go get安装hugo的坑、go module、goinstall、gobuild。"
date: 2020-12-08T21:44:29+08:00
categories: ["code"]
tags: ["golang", "hugo"]
draft: false
---

> 该文章的golang版本为`go version go1.14.5 darwin/amd64`

## 心路历程

### 1. 报错

晚上本来想装个hugo玩玩，hugo提供了挺多安装方法比如最直接的二进制包安装、homebrew等包管理安装、docker，正好电脑里也有golang的环境，平时装go写的软件也很方便`go get`就解决了，自然而然想到`go get -v github.com/gohugoio/hugo`，看着控制台Download了半天最后竟然给我报了个错：

```cmd
package github.com/jdkato/prose/transform: cannot find package "github.com/jdkato/prose/transform" in any of:
        /usr/local/Cellar/go/1.14.5/libexec/src/github.com/jdkato/prose/transform (from $GOROOT)
        /Users/jabin/go/src/github.com/jdkato/prose/transform (from $GOPATH)
```

大概意思就是找不到`github.com/jdkato/prose/transform`这个包，我下意识点开[https://github.com/jdkato/prose/transform](https://github.com/jdkato/prose/transform)一探究竟，发现github确实报了404错误，说明这个包确实有点问题。

### 2. 尝试

然后我寻思着不对劲怎么装都装不上，打开[hugo的github仓库](https://github.com/gohugoio/hugo)看到了这样的向导：

```cmd
mkdir $HOME/src
cd $HOME/src
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install
```

我不信邪，就试了一下先`clone`再`go install`会怎么样，然后发现竟然成功了，在golang的bin目录（在我的电脑下是`~/go/bin`）下生成了一个名为`hugo`的可执行文件。

### 3. 排错

我寻思着`go get`是把项目先拉下来然后编译放在`~/go/bin`下，`go install`是自己手动进入了某个项目目录下执行编译输出到`~/go/bin`，二者应该只有**自己把项目拉下来然后进入该项目**的区别，怎么会出现这样奇怪的错误。

#### 3.1 灵异事件

一遍思考一遍手上又执行了一遍`go get -v github.com/gohugoio/hugo`，这一遍竟然什么报错也没有，我赶紧删掉`~/go/bin`下的编译产物`hugo`，检验是否真的`go get`能够正常编译出可执行文件了，结果是**确实没有报错且正常编译出结果了**。

我百思不得其解，但是回想一开始得到的错误，似乎错误中提到了两个路径：

- `/usr/local/Cellar/go/1.14.5/libexec/src/github.com/jdkato/prose/transform (from $GOROOT)`
- `/Users/jabin/go/src/github.com/jdkato/prose/transform`
我逐一检查后发现确实两个地址都无法找到需要的这个包，于是我开始想这个项目是不是有什么问题，熟悉go包和github地址的人应该清楚，这个地址说明这个包的作者是`jdkato`，项目名是`prose`，`transform`应该是项目下的一个文件夹。

#### 3.2 计算机不会耍赖皮

> 写文章时[https://github.com/jdkato/prose](https://github.com/jdkato/prose)的最新commit是`c2b2f78b870e41bec89843648b04b1716a0fb9c6`
> [hugo的github仓库](https://github.com/gohugoio/hugo)的最新commit是`c84ad8db821c10225c0e603c6ec920c67b6ce36f`

于是我开始在github搜jdkato这个名字，发现了这个作者确实有prose这个项目[https://github.com/jdkato/prose](https://github.com/jdkato/prose)，点进去一看发现这个项目也的的确确没有`transform`这个文件夹。

再翻看[hugo的github仓库](https://github.com/gohugoio/hugo)发现最新的release是三天前，说明hugo的代码本身没有什么问题。

这时候我得出一种推论，hugo可能用了prose这个项目的老版本，老版本里存在transform这个文件夹，hugo可能是自己的golang环境里缓存了prose这个项目的老版本之类的。但是很快这种想法就被我自己推翻了，因为我的的确确在自己的电脑上`go install`成功了hugo的最新源码，并且在`go install`成功之前我还处于`package github.com/jdkato/prose/transform: cannot find`的错误中，根本不存在缓存了老版本项目代码的可能。

#### 3.3 真相只有一个

推测到这里，我想起来一个重要的东西，`go.mod`正是那个**可以缓存旧版本的**关键人物啊！我赶紧再翻看[hugo的github仓库](https://github.com/gohugoio/hugo)，果然发现了`go.mod`这个文件，里面也有这样一段记录：

```go
module github.com/gohugoio/hugo

require (
    ...省略
    github.com/jdkato/prose v1.2.0
    ...省略
)

replace github.com/markbates/inflect => github.com/markbates/inflect v0.0.0-20171215194931-a12c3aec81a6

go 1.12
```

可以看到`require`中指出了对项目`github.com/jdkato/prose`的版本要求为`v1.2.0`，我啪的一下就打开了[https://github.com/jdkato/prose/tree/v1.2.0](https://github.com/jdkato/prose/tree/v1.2.0)，果然在`v1.2.0`这个旧版本的`tag`下有我们想要的`transform`文件夹。

那么`go get`报错的真相就只有一个，`go get`获取了最新版本的`github.com/jdkato/prose`，正好最新版本的`github.com/jdkato/prose`重构了，项目结构发生了改变。而`go install`成功后`go get`也能成功的灵异事件也能说的过去了，就是因为`go install`获取到了对的版本的项目，导致`go get`也能够找到需要的文件了。

#### 3.4 案件远没有结束

尽管得出了一个看起来说的过去的解释，但是问题又来了：**为什么`go install`就能获取到对应的版本的项目？**

我自然而然想到了`go.mod`这个关键人物，我同样得出一个大胆的推论：`go get`不会理会`go.mod`中的限制，而`go install`则会在意。

#### 3.5 口说无凭，怎么证明

粗略了翻阅了有关`go get`和`go install`的一些说明，发现没什么和我这个问题相关的，只好来点硬核的了，正好go也是自举的，go语言的go语言源码还是可以看看的。

#### 3.6 源码中的线索

```cmd
git clone https://github.com/golang/go.git
cd go
code .
```

上来直接一个闪电三连码，用命令行打开我习惯的vscode开始看。

通过搜索找到了`go get`和`go install`的源码，分别位于

|命令|源码位置|
|-|-|
|go get|./go/internal/get/get.go|
|go install|./go/internal/work/build.go|

**为什么`go install`不是在install.go里？**根据我的观察，因为go install和go build的功能和实现都很接近，所以这两个命令的源码都在/go/internal/work/build.go。

主要看看`get.go`的源码，里面有一个`runGet`方法，这是执行`go get`的入口方法，该方法中存在这样的调用链：
`load.PackagesForBuild`->`PackagesAndErrors`->`ImportPaths`

|方法名|源码位置|
|-|-|
|load.PackagesForBuild|.go/internal/load/pkg.go|
|PackagesAndErrors|.go/internal/load/pkg.go|
|ImportPaths|.go/internal/load/pkg.go|

这个`ImportPaths`方法完整如下：

```go
func ImportPaths(args []string) []*search.Match {
    if ModInit(); cfg.ModulesEnabled {
        return ModImportPaths(args)
    }
    return search.ImportPaths(args)
}
```

可一看到源码中关于`ModInit(); cfg.ModulesEnabled`是否成立有两种处理模式。

#### 3.7 关键信息

因为源码里对于`ModInit()`只是简单的`var ModInit func()`，没有方法体可以看到，也许是再别的地方进行了实现，我也没有深究。主要是看到了`cfg.ModulesEnabled`这个变量，直接想到了go module是否生效的问题，于是百度了`go module`并且看到了这篇文章`[go module 基本使用](https://www.cnblogs.com/chnmig/p/11806609.html)`，里面有这样一段话：

> go在1.13版本默认是auto,~~代表 当项目在 GOPATH/src 外且项目根目录有 go.mod 文件时，开启 go module.也就是说,如果你不把代码放置在 GOPATH/src 下则默认使用 MODULE 管理.~~
不好意思看错了,1.13+的版本判断开不开启MODULE的依据是根目录下有没有go.mod文件
我们也可手动更改为 on(全部开启)/off(全部不开启)

我恍然大悟，因为我`go get`的地方正好就是随便一个地方->没有在项目里->自然没有`go.mod`->也就没有开启`module`->`ModInit(); cfg.ModulesEnabled`不成立->`go get`源码进入了另一种处理方式->hugo源码中的`go.mod`没有生效，一切都解释的通了。

#### 3.8 验证

验证的方法也很简单，根据前文的判断关键，新建一个有`go.mod`的环境再执行一次`go get`就可以了。

```cmd
mkdir testgo
cd testgo 
go mod init xxx
go: creating new go.mod: module xxx
go get -v github.com/gohugoio/hugo
```

在有`go.mod`存在的环境下是可以安装成功的，也证明了前面的说法，至此一切都解决了。

### 4. 结尾

这次的问题也花了一整个下午的时间去排查，先是花了一些时间思考可能的原因，又花了很长时间去看源码，读源码真的花了很长很长的时间，一连看了几个小时直到头都有些晕了在起身去吃饭，吃完饭回来有精神了继续看了一会就发现问题了，可能在文章中就是简单的调用链路和几行关键代码，但是怎么在上千行的源码中找到这些关键信息远没有文章中的那么简单，首先是要看懂，然后是顺着调用链往下继续看懂，有时候还会误解然后找错方向找了很久……

但结果还是非常好的，解决了自己的疑惑，更难能的是在源码里学到了几个很有意思很妙的写法，比如这两个：

```go
// 这个else if的顺序和作用域利用的很充分
func CleanPatterns(patterns []string) []string {
    // 省略
    var out []string
    for _, a := range patterns {
        var p, v string
        if build.IsLocalImport(a) || filepath.IsAbs(a) {
            p = a
        } else if i := strings.IndexByte(a, '@'); i < 0 {
            p = a
        } else {
            p = a[:i]
            v = a[i:]
        }
        // 省略
    }
    // 省略
}
```

```go
// 匿名函数+函数变量+递归+闭包的妙用
func PackageList(roots []*Package) []*Package {
    seen := map[*Package]bool{}
    all := []*Package{}
    var walk func(*Package)
    walk = func(p *Package) {
        if seen[p] {
            return
        }
        seen[p] = true
        for _, p1 := range p.Internal.Imports {
            walk(p1)
        }
        all = append(all, p)
    }
    for _, root := range roots {
        walk(root)
    }
    return all
}
```

写的有点长了，但还是想完整的记录一下心路历程，折腾的过程还是很有意思的。
