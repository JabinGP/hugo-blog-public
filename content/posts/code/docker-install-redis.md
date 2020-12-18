---
title: "docker安装redis、redis-cli测试连接、python api测试连接。"
date: 2020-12-08T21:41:35+08:00
categories: ["code"]
tags: ["docker", "redis", "python", "redis-cli"]
draft: false
---

## 拉取镜像、启动

拉取镜像

```cmd
docker pull redis
```

查看已有镜像

```cmd
docker images
```

启动 docker

```cmd
docker run -itd --name redis -p 6379:6379 redis
```

查看 docker 运行的容器状态

```cmd
docker ps
```

## 测试使用

### 在容器内部测试连接

```cmd
docker exec -it redis /bin/bash
```

进入容器内部后

```bash
root@2ff6e2c47742:/data# redis-cli
127.0.0.1:6379> set username jabin
OK
127.0.0.1:6379> get username
"jabin"
127.0.0.1:6379>
```

### 在容器外部测试连接

redis 容器绑定到了宿主主机的**6379**端口，因此可以认为 redis 跑在了本机的**6379**端口。

#### 使用 redis-cli 连接

如果本机（宿主机）安装了**redis-cli**，可以使用本机的**redis-cli**测试。

```bash
➜  Desktop redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> get username
"jabin"
127.0.0.1:6379>
```

#### 使用 python 的 api 连接

```py
import redis

r = redis.StrictRedis(host='127.0.0.1', port=6379)

print(str(r.get("username"), encoding="utf-8"))
```

输出

```cmd
jabin
```
