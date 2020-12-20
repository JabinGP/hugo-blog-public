---
title: "CentOS使用Yum升级Git到2.1x新版本"
date: 2020-12-08T21:13:54+08:00
categories: ["code"]
tags: ["linux", "centos", "git", "update", "yum"]
draft: false
---

使用yum最多只能安装到1.8，版本太旧了，下载源码手动编译安装？先不说国内下载官网包2kB/s的速度，就是下载下来了编译也麻烦啊，包管理是吃干饭的嘛？

其实只要换个源，重新下载就好了

先卸载旧版

```cmd
yum remove git
```

添加新源后安装新版

```cmd
yum install -y https://centos7.iuscommunity.org/ius-release.rpm
yum install -y git2u
```

检验

```cmd
git version    
```

最后附带我的安装过程

```cmd
root@izwz957qhjacaocedubzjjz /tmp/installGit   [20:43:53] 
> # yum install -y https://centos7.iuscommunity.org/ius-release.rpm
已加载插件：fastestmirror
ius-release.rpm                    | 8.2 kB     00:00     
正在检查 /var/tmp/yum-root-6VAioA/ius-release.rpm: ius-release-2-1.el7.ius.noarch
/var/tmp/yum-root-6VAioA/ius-release.rpm 将被安装
正在解决依赖关系
--> 正在检查事务
---> 软件包 ius-release.noarch.0.2-1.el7.ius 将被 安装
--> 正在处理依赖关系 epel-release = 7，它被软件包 ius-release-2-1.el7.ius.noarch 需要
Loading mirror speeds from cached hostfile
--> 正在检查事务
---> 软件包 epel-release.noarch.0.7-12 将被 安装
--> 解决依赖关系完成

依赖关系解决

==========================================================
 Package       架构    版本           源             大小
==========================================================
正在安装:
 ius-release   noarch  2-1.el7.ius    /ius-release  4.5 k
为依赖而安装:
 epel-release  noarch  7-12           epel           15 k

事务概要
==========================================================
安装  1 软件包 (+1 依赖软件包)

总计：19 k
总下载量：15 k
安装大小：29 k
Downloading packages:
epel-release-7-12.noarch.rpm         |  15 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : epel-release-7-12.noarch              1/2 
警告：/etc/yum.repos.d/epel.repo 已建立为 /etc/yum.repos.d/epel.repo.rpmnew 
  正在安装    : ius-release-2-1.el7.ius.noarch        2/2 
  验证中      : ius-release-2-1.el7.ius.noarch        1/2 
  验证中      : epel-release-7-12.noarch              2/2 

已安装:
  ius-release.noarch 0:2-1.el7.ius                        

作为依赖被安装:
  epel-release.noarch 0:7-12                              

完毕！
                                                          
root@izwz957qhjacaocedubzjjz /tmp/installGit   [20:44:05] 
> # yum install -y git2u                                 
已加载插件：fastestmirror
ius                                | 1.3 kB     00:00     
ius/x86_64/primary                   | 129 kB   00:01     
Loading mirror speeds from cached hostfile
ius                                               538/538
正在解决依赖关系
--> 正在检查事务
---> 软件包 git2u.x86_64.0.2.16.5-1.ius.el7 将被 安装
--> 正在处理依赖关系 git2u-perl-Git = 2.16.5-1.ius.el7，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在处理依赖关系 git2u-core-doc = 2.16.5-1.ius.el7，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在处理依赖关系 git2u-core = 2.16.5-1.ius.el7，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在处理依赖关系 perl(Git::I18N)，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在处理依赖关系 perl(Git)，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在处理依赖关系 libsecret-1.so.0()(64bit)，它被软件包 git2u-2.16.5-1.ius.el7.x86_64 需要
--> 正在检查事务
---> 软件包 git2u-core.x86_64.0.2.16.5-1.ius.el7 将被 安装
---> 软件包 git2u-core-doc.noarch.0.2.16.5-1.ius.el7 将被 安装
---> 软件包 git2u-perl-Git.noarch.0.2.16.5-1.ius.el7 将被 安装
---> 软件包 libsecret.x86_64.0.0.18.6-1.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

==========================================================
 Package         架构    版本                 源     大小
==========================================================
正在安装:
 git2u           x86_64  2.16.5-1.ius.el7     ius   1.1 M
为依赖而安装:
 git2u-core      x86_64  2.16.5-1.ius.el7     ius   5.5 M
 git2u-core-doc  noarch  2.16.5-1.ius.el7     ius   2.4 M
 git2u-perl-Git  noarch  2.16.5-1.ius.el7     ius    68 k
 libsecret       x86_64  0.18.6-1.el7         base  153 k

事务概要
==========================================================
安装  1 软件包 (+4 依赖软件包)

总下载量：9.2 M
安装大小：42 M
Downloading packages:
警告：/var/cache/yum/x86_64/7/ius/packages/git2u-2.16.5-1.ius.el7.x86_64.rpm: 头V4 RSA/SHA256 Signature, 密钥 ID 4b274df2: NOKEY
git2u-2.16.5-1.ius.el7.x86_64.rpm 的公钥尚未安装
(1/5): git2u-2.16.5-1.ius.el7.x86_64 | 1.1 MB   00:02     
(2/5): git2u-core-doc-2.16.5-1.ius.e | 2.4 MB   00:00     
(3/5): git2u-core-2.16.5-1.ius.el7.x | 5.5 MB   00:03     
(4/5): libsecret-0.18.6-1.el7.x86_64 | 153 kB   00:00     
(5/5): git2u-perl-Git-2.16.5-1.ius.e |  68 kB   00:00     
----------------------------------------------------------
总计                         2.6 MB/s | 9.2 MB  00:03     
从 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-IUS-7 检索密钥
导入 GPG key 0x4B274DF2:
 用户ID     : "IUS (7) <dev@ius.io>"
 指纹       : c958 7a09 a11f d706 4f0c a0f4 e558 0725 4b27 4df2
 软件包     : ius-release-2-1.el7.ius.noarch (installed)
 来自       : /etc/pki/rpm-gpg/RPM-GPG-KEY-IUS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : git2u-core-2.16.5-1.ius.el7.x86_64    1/5 
  正在安装    : git2u-core-doc-2.16.5-1.ius.el7.noa   2/5 
  正在安装    : libsecret-0.18.6-1.el7.x86_64         3/5 
  正在安装    : git2u-perl-Git-2.16.5-1.ius.el7.noa   4/5 
  正在安装    : git2u-2.16.5-1.ius.el7.x86_64         5/5 
  验证中      : git2u-2.16.5-1.ius.el7.x86_64         1/5 
  验证中      : git2u-core-doc-2.16.5-1.ius.el7.noa   2/5 
  验证中      : git2u-core-2.16.5-1.ius.el7.x86_64    3/5 
  验证中      : git2u-perl-Git-2.16.5-1.ius.el7.noa   4/5 
  验证中      : libsecret-0.18.6-1.el7.x86_64         5/5 

已安装:
  git2u.x86_64 0:2.16.5-1.ius.el7                         

作为依赖被安装:
  git2u-core.x86_64 0:2.16.5-1.ius.el7                    
  git2u-core-doc.noarch 0:2.16.5-1.ius.el7                
  git2u-perl-Git.noarch 0:2.16.5-1.ius.el7                
  libsecret.x86_64 0:0.18.6-1.el7                         

完毕！
                                                          
root@izwz957qhjacaocedubzjjz /tmp/installGit   [20:44:22] 
> # git version                                       

git version 2.16.5
```

> 感谢[https://www.cnblogs.com/jhxxb/p/10571227.html](https://www.cnblogs.com/jhxxb/p/10571227.html)
