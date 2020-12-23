---
title: "有关Linux中的Service"
date: 2020-12-22T15:18:25+08:00
draft: true
---

> 本文不会涉及很多专业的表达，因为我并不喜欢那些晦涩难懂的定义，并且事实上我也记不住那些定义。。。

## 1. 我说的Service是什么

> 我假设你已经接触过Apache、Nginx这类HTTP服务器，又或者是MySQL、MongoDB、Redis这类数据库。

无论是**HTTP服务器**又或者是**数据库**都有一个共同的特点，就是**默默地运行**在**后台**等待你的请求并给予你响应，我称这样的程序为**Service(服务)**。

## 2. 前台与后台

> 以下提到的命令需要在linux环境执行。

### 2.1. 前台

当你运行一个前台程序`ping`时:

```cmd
root@jabingp /etc                                                                     [16:52:07] 
> # ping google.com                                                                                             
PING google.com (216.58.200.46) 56(84) bytes of data.
```

你会惊奇的发现你得不到任何的回应，因为在国内你的网络环境确确实实访问不到`google`。

这时候你感到非常生气，你想直接关掉这个该死的程序，于是你按下了`Ctrl`+`C`_（这里通过组合键发出了一个关闭信号，ping接收到这个信号后就关闭了）_，这该死的程序终于**停止了**。

```cmd
root@jabingp /etc                                                                     [16:57:18] 
> # ping google.com                                                                                             
PING google.com (93.46.8.90) 56(84) bytes of data.
^C
--- google.com ping statistics ---
19 packets transmitted, 0 received, 100% packet loss, time 17999ms
```

### 2.2. 后台

但当你想关闭运行在后台的服务时，就不是按个组合键那么简单了。假如我们把上面的`ping`命令放到后台来执行，在`linux`的命令行里你只需要加上`&`就可以，这里我们来ping一个有回应的网站，同时我们指定一下ping的**间隔为10秒**一次，不要让它刷屏了。

先来不后台运行：

```cmd
root@jabingp /tmp                                                                     [17:25:59] 
> # ping baidu.com -i 10                                                                                        
PING baidu.com (39.156.69.79) 56(84) bytes of data.
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=1 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=2 ttl=49 time=39.9 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=3 ttl=49 time=39.9 ms
```

现在加上`&`后台运行：

```cmd
root@izwz98zwp24pi5mjf0bo9az /tmp                                                                     [17:30:27] 
> # ping baidu.com -i 10 &                                                                                      
[1] 28786
PING baidu.com (39.156.69.79) 56(84) bytes of data.                                                              

root@izwz98zwp24pi5mjf0bo9az /tmp                                                                     [17:30:29] 
> # 64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=1 ttl=49 time=40.0 ms                                   
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=2 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=3 ttl=49 time=39.9 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=4 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=5 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=6 ttl=49 time=39.9 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=7 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=8 ttl=49 time=40.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=9 ttl=49 time=39.9 ms
```

你会发现无论你怎么`Ctrl`+`C`都无法关闭ping这个程序了，因为它已经在后台运行了。并且还会将ping的结果不断的输出到控制台上，让你的控制台看起来很错乱。

你现在十分不爽，你想赶紧把这个扰乱你视线的ping关闭，但是组合键已经失效了，你无能为力。

虽然现在不能直接通过组合键发出信号让ping关闭，但是借助操作系统内置的进程管理我们还是可以找到这个烦人的东西并关掉的，执行命令`ps -aux | grep ping`：

```cmd
ps -aux | grep ping
root     28786  0.0  0.1 147820  1996 pts/0    SN   17:30   0:00 ping baidu.com -i 10
```

可以在第二行的结果中看到了我们刚刚敲过的`ping baidu.com -i 10`命令，并且留意`28786`这个数字，这个是`进程id`，也就是这个进程的**身份证**，有了这个我们就可以把这个烦人的东西关掉了。不用理会控制台上的输出，只需要输入或者复制`kill -9 28786`并且回车

```cmd
kill -9 28786
[1]  + 28786 killed     ping baidu.com -i 10   
```

你会发现这个烦人的东西已经被关掉了。

### 2.3 总结

通过前面两个例子可以看到，当程序在前台运行的时候，我们使用组合键`Ctrl`+`C`就可以发出关闭信号使得程序关闭。但是当程序到了后台运行时，我们想要关闭这个进程就需要借助操作系统的`ps`命令先获取这个**进程**的`id`，再通过`kill`命令去向这个`id`的进程发出关闭信号才能得它关闭。

## 3. 愚蠢的做法

想象一下，当你的linux上运行了无数种后台程序：Apache、Nginx、MySQL、MongoDB、Redis...

这时候你想要关掉这里面的某些软件，或者是你仅仅只是修改了配置文件而需要重启一下这些程序，按照上面的做法你可能需要先找到进程的id、再发出关闭信号、再找找用什么命令能够启动，很麻烦对吧？你仅仅只是想重启一下而已。

## 4. 聪明的做法

> 在一些旧版本的linux中使用init.d和service来完成

有没有什么可以帮我处理好这些繁琐的流程呢？

幸运的是在linux里你可以借助`Systemd`来完成这些事情，前提是你告诉Systemd该如何去管理你的这些程序，也就是用规定的格式去编写一些配置信息。

更幸运的是，如果你通过linux自带的包管理如`yum`、`apt`、`pacman`来安装软件，例如借助命令`yum install nginx`来安装Nginx，包管理软件已经自动帮你完成了Systemd有关配置信息的编写，你只需要知道包管理为你配置的服务**叫什么名字**就可以。

> 留意！并不是所有的软件配置名称都相同。

显然对于Nginx而言，配置的名称就叫做`nginx`，因此你可以这样对Nginx操作：

- 启动：`sudo systemctl start nginx`
- 重启：`sudo systemctl restart nginx`
- 关闭：`sudo systemctl stop nginx`

当你仅仅是修改了Nginx的配置文件需要重启时，你只需要一条命令就可以完成那些繁琐的过程，针不戳。

## 你可以在哪些地方找到Service

## 自己动手配置Service

## 参考文章

- [https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap3sec21.html](https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap3sec21.html)
- [http://www.cnitblog.com/luckydmz/archive/2020/06/28/92250.html](http://www.cnitblog.com/luckydmz/archive/2020/06/28/92250.html)
- [http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
- [http://www.ruanyifeng.com/blog/2016/02/linux-daemon.html](http://www.ruanyifeng.com/blog/2016/02/linux-daemon.html)
- [https://tldp.org/LDP/sag/html/major-services.html](https://tldp.org/LDP/sag/html/major-services.html)
- [https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap3sec21.html](https://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/chap3sec21.html)
