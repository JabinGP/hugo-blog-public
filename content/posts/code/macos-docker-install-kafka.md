---
title: "macOS下使用docker安装kafka、遇到的坑与解决方案。"
date: 2020-12-08T21:39:49+08:00
images: []
categories: ["code"]
tags: ["macos", "kafka", "zookeeper", "docker"]
draft: false
---

由于`kafka`依赖`zookeeper`，因此需要使用 docker 同时安装`zookeeper`和 `kafka`。

坑：由于 macOS 的 docker 底层实现的不同，网上的很多教程放在 macOS 中并不能成功运行，主要原因是 macOS 的 docker 在容器和宿主之间无法通过 ip 直接通信。因此在安装的时候需要特殊注意与 ip 相关的设置，当容器需要访问宿主ip时，需要使用`docker.for.mac.host.internal`或者`host.docker.internal`代替。

## 拉取镜像

```cmd
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
```

## 启动容器

### 启动 zookeeper

```cmd
docker run -d --name zookeeper -p 2181:2181 wurstmeister/zookeeper
```

- `-d` 参数设置后台运行
- `--name zookeeper` 参数指定容器别名
- `-p 2181:2181` 参数绑定容器端口到宿主端口

### 启动 kafka

注意，**kafka 依赖 zookeeper**，启动 kafka 前需要先启动 zookeeper。

以下配置默认 kafka 端口配置在 **9092** 端口。

```cmd
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT={host-ip}:{zookeeper-port}/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://{host-ip}:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka
```

`-d` 参数指定容器后台运行

`--name kafka` 参数指定容器别名

`-e` 参数可以设置 docker 容器内环境变量，每个变量的解释：

- `KAFKA_BROKER_ID=0`
  在 kafka 集群中，每个 kafka 都有一个 BROKER_ID 来区分自己
- `KAFKA_ZOOKEEPER_CONNECT={host-ip}:{zookeeper-port}/kafka`
  配置 zookeeper 管理 kafka 的路径
- `KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://{host-ip}:9092`
  把 kafka 的地址端口注册给 zookeeper
- `KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092`
  kafka 监听地址

比如我的电脑是 mac，在 host-ip 这块就不能填本机 ip（windows 和 linux 可以），需要填`docker.for.mac.host.internal`，zookeeper 端口启动在**2181**，kafka 即将启动在**9092**，那么我的命令是这样的`docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=docker.for.mac.host.internal:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://docker.for.mac.host.internal:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka`

## 测试功能是否正常

### 测试 kafka 生产与消费

#### 进入 kafka 容器

```cmd
docker exec -it kafka bash
```

#### 进入 kafka 容器中的脚本目录

注意，此时应该已经进入到了容器中的`bash`。

进入 kafka 的脚本目录，其中`kafka_2.12-2.5.0`可能会随着版本而变化数字。

```cmd
cd /opt/kafka_2.12-2.5.0/bin
```

通过 ls 可以看到许多的.sh 脚本

```cmd
bash-4.4# ls
connect-distributed.sh               kafka-delete-records.sh              kafka-server-stop.sh
connect-mirror-maker.sh              kafka-dump-log.sh                    kafka-streams-application-reset.sh
connect-standalone.sh                kafka-leader-election.sh             kafka-topics.sh
kafka-acls.sh                        kafka-log-dirs.sh                    kafka-verifiable-consumer.sh
kafka-broker-api-versions.sh         kafka-mirror-maker.sh                kafka-verifiable-producer.sh
kafka-configs.sh                     kafka-preferred-replica-election.sh  trogdor.sh
kafka-console-consumer.sh            kafka-producer-perf-test.sh          windows
kafka-console-producer.sh            kafka-reassign-partitions.sh         zookeeper-security-migration.sh
kafka-consumer-groups.sh             kafka-replica-verification.sh        zookeeper-server-start.sh
kafka-consumer-perf-test.sh          kafka-run-class.sh                   zookeeper-server-stop.sh
kafka-delegation-tokens.sh           kafka-server-start.sh                zookeeper-shell.sh
bash-4.4#
```

#### 测试 kafka 生产者和消费者

##### 启动 kafka 生产者

运行 kafka 生产者发送消息

```cmd
./kafka-console-producer.sh --broker-list localhost:9092 --topic first-topic
```

看到出现了个对话提示的小`>`就可以发送消息了，不过不要着急，先把消费者启动了。

##### 启动 kafka 消费者

另起一个终端，进入 kafka 容器，进入`/opt/kafka_2.12-2.5.0/bin`目录，运行 kafka 消费者接收消息

```cmd
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first-topic --from-beginning
```

##### 测试发送和接收消息

在生产者中发送`Hello`，消费者中应该能够收到`Hello`。

生产者

```cmd
jabin@jabindeiMac ~/code/tmp » docker exec -it kafka bash
bash-4.4# cd /opt/kafka_2.12-2.5.0/bin/
bash-4.4# ./kafka-console-producer.sh --broker-list localhost:9092 --topic first-topic
>Hello
>
```

消费者

```cmd
jabin@jabindeiMac ~/code/tmp » docker exec -it kafka bash
bash-4.4# cd /opt/kafka_2.12-2.5.0/bin/
bash-4.4# ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first-topic --from-beginning
Hello
```

## 参考文章

[https://www.jianshu.com/p/e8c29cba9fae](https://www.jianshu.com/p/e8c29cba9fae)
[https://docs.docker.com/docker-for-mac/networking/](https://docs.docker.com/docker-for-mac/networking/)
[https://yuanmomo.net/2019/06/13/docker-network/](https://yuanmomo.net/2019/06/13/docker-network/)
[https://www.jianshu.com/p/052f9c6ca664](https://www.jianshu.com/p/052f9c6ca664)
