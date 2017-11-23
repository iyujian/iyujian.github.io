---
layout: post
title:  "Spring Cloud Eureka 入门 - Spring Cloud"
date:   2017-11-22 15:31:58 +0800
categories: java
tags: [Spring Cloud]
---

Eureka 是一个提供服务注册与发现的组件（官方文档地址： https://springcloud.cc/spring-cloud-dalston.html#spring-cloud-eureka-server）。

> 1、Eureka server 提供服务注册与发现服务

创建Spring boot web应用，pom.xml添加Eureka server依赖：

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
{% endhighlight %}

在Spring boot启动类添加注解“@EnableEurekaServer”。

{% highlight java %}
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
{% endhighlight %}

修改配置文件“application.yml”：

```
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  server:
    enableSelfPreservation: false
    evictionIntervalTimerInMs: 5000
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

启动工程，输入http://localhost:8761即可看到Eureka Server首页。

> 2、Eureka client： 服务提供者和服务消费者

创建Spring boot 应用，pom.xml添加Eureka依赖：

{% highlight xml %}
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
{% endhighlight %}

Spring boot 启动类添加注解“@EnableEurekaClient”或者“@EnableDiscoveryClient”。

{% highlight java %}
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class EurekaClientApplication {
	public static void main(String[] args) {
		SpringApplication.run(EurekaClientApplication.class, args);
	}

	@GetMapping("/hello")
    public String serviceUrl() {
       return "Hello eureka.";
    }
}
{% endhighlight %}

修改配置文件“application.yml”：

```
server:
  port: 8762
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  	healthcheck:
	  enabled: true
  instance:
    prefer-ip-address: true
spring:
  application:
    name: service-provider
```

运行EurekaClientApplication，浏览器打开http://localhost:8761，会看到EUREKA-CLIENT已经注册到Eureka server上了。

如何消费服务？消费者也是一个Eureka client，配置同上。

{% highlight java %}
@Autowired
private EurekaClient eurekaClient;

@GetMapping("/")
public String hello() {
    return restTemplate.getForObject(eurekaClient.getNextServerFromEureka("SERVICE-PROVIDER", false).getHomePageUrl() + "/hello", String.class);
}
{% endhighlight %}