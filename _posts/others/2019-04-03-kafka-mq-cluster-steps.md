---
layout: post
title:  "Kafka MQ 集群配置步骤"
date:   2019-04-03 17:15:34 +0800
categories: others
tags: [Kafka]
---
以集群中有三个节点为例：

## 1、下载Kafka并解压
```text
> cd /opt
> curl http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.2.0/kafka_2.12-2.2.0.tgz
> tar -xzf kafka_2.12-2.2.0.tgz
> cd kafka_2.12-2.2.0
```

## 2、开启Zookeeper
### 2.1 方法一：Kafka的安装包中自带了Zookeeper，可以直接启动：
```text
> bin/zookeeper-server-start.sh config/zookeeper.properties
```
### 2.2 方法二（推荐）：自建Zookeeper集群，参考：[Zookeeper集群配置](http://www.iyujian.me/others/zookeeper-cluster-steps.html "Zookeeper集群配置") 

## 3、修改各节点的 `server.properties`
```text
> vi config/server.properties
```
### 3.1 修改 `log.dirs` 为 `/data/kafka-logs`，防止临时目录被清理
```properties
# log.dirs=/tmp/kafka-logs 
log.dirs=/data/kafka-logs
```
### 3.2 修改 `broker.id` 分别为 `0,1,2`，保证在集群中不重复
```properties
broker.id=0
```
### 3.3 修改 `zookeeper.connect` 指向Zookeeper集群的IP和端口
```properties
zookeeper.connect=20.1.1.125:2181,20.1.1.126:2181,20.1.1.127:2181
```
### 3.4 将 `listeners` 配置为本机的IP和Kafka端口
```properties
#listeners=PLAINTEXT://:9092
listeners=PLAINTEXT://20.1.1.125:9092
```
### 3.5 `advertised.listeners`(可选配置)，暴露给外部的listeners，若不配置默认使用 listeners 的配置。
```properties
advertised.listeners=PLAINTEXT://20.1.1.125:9092
```
## 4、常规命令
```text
# 启动，-daemon 表示后台启动
> sh ./bin/kafka-server-start.sh -daemon ./config/server.properties
# 停止
> sh ./bin/kafka-server-stop.sh
# 创建 topic
> bin/kafka-topics.sh --create --zookeeper 20.1.1.125:2181,20.1.1.126:2181,20.1.1.127:2181 --replication-factor 3 --partitions 3 --topic my-replicated-topic
# 查看 topic 的信息 
> bin/kafka-topics.sh --describe --zookeeper 20.1.1.125:2181,20.1.1.126:2181,20.1.1.127:2181 --topic my-replicated-topic
# 发送消息
> bin/kafka-console-producer.sh --broker-list 20.1.1.125:9092,20.1.1.126:9092,20.1.1.127:9092 --topic my-replicated-topic
# 消费消息
> bin/kafka-console-consumer.sh --bootstrap-server 20.1.1.125:9092,20.1.1.126:9092,20.1.1.127:9092 --from-beginning --topic my-replicated-topic
```