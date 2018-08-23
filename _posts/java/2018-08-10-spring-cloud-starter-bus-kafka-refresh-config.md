---
layout: post
title:  "Spring Cloud Bus Kafka + Config 刷新集群配置 - Spring Cloud"
date:   2018-08-10 17:39:46 +0800
categories: java
tags: [Spring Cloud]
---

用Spring cloud config搭建了配置中心后，当修改配置后，可以通过直接发送post请求到[/actuator/refresh]进行刷新配置，无需重启服务新配置即可生效。

而当我们的服务部署了集群后，就需要对每个节点挨个发送请求进行刷新，影响维护效率，消息总线 Spring Cloud Bus 可以帮助解决这个问题。

前置条件：

a. 启动Spring cloud eureka server： http://localhost:8888;

b. 启动Spring cloud config server;

c. 启动kafka，端口：9092。

1、首先创建springboot工程，pom.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-kafka</artifactId>
		</dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```
2、bootstrap.yml中加入config client及euerka client配置

```
spring:
  cloud:
    config:
      name: demo
      profile: development
      label: master
      discovery:
        serviceId: config-server
        enabled: true
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/
```

3、application.yml中加入kafka相关配置:

```
spring:
  kafka:
    bootstrap-servers: 127.0.0.1:9092
    consumer:
      group-id: test-consumer-group
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

4、启动类

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
@EnableDiscoveryClient
@RefreshScope
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}


	@Value("${foo}")
    private String foo;

    @GetMapping("get")
    public String get() {
        return foo;
    }

}

```

5、测试

启动两个实例，端口分别是8080和8081。

发送更新命令同时更新2个实例：
curl -X POST http://localhost:8080/actuator/bus-refresh

指定刷新范围： 
curl -X POST http://localhost:8080/actuator/bus-refresh?destination=demo
curl -X POST http://localhost:8080/actuator/bus-refresh?destination=demo:8080

6、进一步优化

既然Spring cloud bus 的 /actuator/bus-refresh 接口提供了针对服务和实例进行配置更新的参数，那么我们就可以在Spring config server中也引入Spring cloud bus, 一起加入到消息总线中来。然后向config server发送刷新请求并指定刷新范围即可。