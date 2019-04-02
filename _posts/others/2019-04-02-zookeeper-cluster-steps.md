---
layout: post
title:  "Zookeeper集群配置步骤"
date:   2019-04-02 19:28:19 +0800
categories: others
tags: [Zookeeper]
---
zookeeper集群
1、下载并安装jdk
2、下载并解压zookeeper-3.4.13.tar.gz
```text
cd /opt
curl https://archive.apache.org/dist/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
tar -xzvf zookeeper-3.4.13.tar.gz
```
3、复制zoo_sample.cfg为zoo.cfg，zookeeper默认使用zoo.cfg的配置
```text
cp /conf/zoo_sample.cfg zoo.cfg
```
4、修改zoo.cfg
```text
# 修改zk默认的数据存储位置，因为linux的/tmp目录会被清理
dataDir=/tmp/zookeeper > /data/zookeeper
# 文件最后加上集群各个节点的信息，格式为：
# server.x=ip:portA:portB
# x: 集群中节点的标识
# ip: 集群中节点的ip
# portA: 与Leader通信的端口，端口可以自定义，不重复就行
# portB: 用于选举的端口，端口可以自定义，不重复就行
server.1=20.1.1.125:2888:3888
server.2=20.1.1.126:2888:3888
server.3=20.1.1.127:2888:3888
```
5、在上面配置的dataDir目录创建文件，文件名为：myid，文件内容分别为1，2，3（与上面的x对应）
```text
cd /data
mkdir zookeeper
echo 1 > myid
```
6、常规命令
```text
#启动：
sh bin/zkServer.sh start
#停止：
sh bin/zkServer.sh stop
#查看状态：
sh bin/zkServer.sh status
#打开客户端：
sh bin/zkCli.sh
```


