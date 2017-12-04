---
layout: post
title:  "Spring Cloud Eureka Server 高可用集群配置 - Spring Cloud"
date:   2017-11-28 16:31:41 +0800
categories: java
tags: [Spring Cloud]
---

Eureka server是没有后端存储的，注册表中的服务实例通过发送心跳包来保持更新（因此可以在内存中完成）。


> 1、独立模式的配置（application.yml）

```
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

> 2、集群部署模式（application.yml）

如果要做集群部署，只需要定义不同的配置文件，通过运行不同配置文件的实例，让各个server之间互相注册。


```
---
server:
  port: 8761
spring:
  profiles: peer1
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8762/eureka/

---
server:
  port: 8762
spring:
  profiles: peer2
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

以上是2个Eureka server实例下的配置，我们可以加更多个实例，只要各个实例之间有一条边可以连接，就可以将服务同步注册到其他实例中去。如果有3个实例（peer1，peer2，peer3），我们可以只需要配置peer1的serviceUrl指向peer2，peer2的指向peer3，peer3的指向peer1即可实现多个实例之间的同步复制。但是这样的配置若有一个实例发生故障，会导致整个集群发生故障(split-brain问题)，这时候我们一般会让peer1同时指向peer2和peer3，peer2指向peer1和peer3，peer3指向peer1和peer2，原则上可以防止split-brain的发生。

> 3、运行

运行“maven clean package”命令将工程打成jar包。使用下面的命令来运行各实例：

```
java -jar eureka-server-0.0.1.jar --spring.profiles.active=peer1
java -jar eureka-server-0.0.1.jar --spring.profiles.active=peer2

```

官方文档： http://cloud.spring.io/spring-cloud-static/Edgware.RELEASE/multi/multi_spring-cloud-eureka-server.html#spring-cloud-eureka-server-zones-and-regions