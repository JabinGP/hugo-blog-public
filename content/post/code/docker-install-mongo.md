---
title: "Docker安装MongoDB"
date: 2020-12-12T17:28:11+08:00
images: []
categories: ["code"]
tags: ["docker", "mongodb"]
authors: []
---

## 拉取镜像

```cmd
docker pull mongo:latest
```

## 运行镜像

```cmd
docker run -itd --name mongo -p 27017:27017 mongo --auth
```

- --auth：Enables authorization to control user’s access to database resources and operations. When authorization is enabled, MongoDB requires all clients to authenticate themselves first in order to determine the access for the client.
Configure users via the mongo shell. If no users exist, the localhost interface will continue to have access to the database until you create the first user。

## 进入容器添加用户

进入容器并且直接运行mongo-shell，选择数据库为`admin`

```cmd
docker exec -it mongo mongo admin
```

创建用户并尝试连接

```cmd
db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
db.auth('admin', '123456')
```

## 验证账户权限

运行一些简单的指令来验证账户的有效性

```cmd
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> show users
{
        "_id" : "admin.admin",
        "userId" : UUID("dc5760ea-c8c1-4f40-af5b-7d9d53779842"),
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```
