---
layout: post
title:  "Spring Cloud Bus Kafka 发送和接受消息 - Spring Cloud"
date:   2018-08-10 10:59:08 +0800
categories: java
tags: [Spring Cloud]
---

在微服务架构的系统中，我们通常会使用轻量级的消息代理来构建一个共用的消息主题让系统中所有微服务实例都连接上来，由于该主题中产生的消息会被所有实例监听和消费，所以我们称它为消息总线。在总线上的各个实例都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息， 例如配置信息的变更或者其他 一 些管理操作等。

由于消息总线在微服务架构系统中被广泛使用，所以它同配置中心一样，几乎是微服务架构中的必备组件。 Spring Cloud 作为微服务架构综合性的解决方案，对此自然也有自己的实现，这就是“Spring Cloud Bus”。 通过使用Spring Cloud Bus,可以非常容易地搭建起消息总线，同时实现了一些消息总线中的常用功能，比如，配合Spring Cloud Config 实现微服务应用配置信息的动态更新等。

目前Spring Cloud Bus提供了对RabbitMQ和Kafka的支持，下面以Kafka为例，介绍一下如何发送和接收消息。

1、首先创建springboot工程，并引入依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-kafka</artifactId>
</dependency>
```

2、application.yml中加入kafka相关配置:

```
spring:
  kafka:
    bootstrap-servers: 127.0.0.1:9092
    producer:
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: test-consumer-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

3、消息生产者

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;

@Component
public class Sender {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 发送消息
     * @param topic
     * @param key
     * @param message
     */
    public void send(String topic, String key, String message) {
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, key, message);
        //发送成功及发送失败的回调
        future.addCallback(result -> logger.debug("发送成功：{}", result), ex -> logger.debug("发送失败：{}", ex));
    }
}

```

4、消息消费者

```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

    private final Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 接受消息
     * @KafkaListener注解的参数“topics”为订阅的主题，与消息发送的主题对应
     * @param record
     */
    @KafkaListener(topics = {"topic_01"})
    public void receive(ConsumerRecord<?, ?> record) {
        logger.debug("key: {}, value: {}, record: {}", record.key(), record.value(), record);
    }

}
```
5、单元测试类
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

	@Autowired
	private Sender sender;

	@Test
	public void send() {
		sender.send("topic_01", "hello", "world.");
	}

}
```