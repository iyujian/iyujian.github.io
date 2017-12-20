---
layout: post
title:  "Spring Cloud Zuul：服务网关之路由服务 - Spring Cloud"
date:   2017-12-20 13:40:40 +0800
categories: java
tags: [Spring Cloud]
---

路由是微服务架构中不可或缺的一部分。Zuul是由Netflix开源的，提供了基于JVM的路由服务和服务端侧负载均衡服务。（官方文档：https://cloud.spring.io/spring-cloud-static/Edgware.RELEASE/single/spring-cloud.html#_router_and_filter_zuul）

## 1. 如何使用Spring Cloud Zuul？

1.1 pom.xml加入spring-cloud-starter-zuul依赖。

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
{% endhighlight %}

1.2 application.yml加入zuul配置

假设现在已有一个用户服务：http://localhost:8081/user/list。

```
zuul:
  routes:
    user:
      path: /user/**
      url: http://localhost:8081/user/
```

1.3 Spring boot启动类加上注解：“@EnableZuulProxy”

{% highlight java %}
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
	public static void main(String[] args) {
		SpringApplication.run(ZuulApplication.class, args);
	}
}
{% endhighlight %}

至此，一个最简单的路由服务配置完成，实现了将请求http://localhost:8080/user/**都转发到http://localhost:8081/user/**。

## 2. 将Zuul服务注册到Eureka server上实现服务化。

2.1. pom.xml在上面的基础上加入eureka client的依赖，并在启动类加上注解“@EnableEurekaClient”。

2.2. 假设上述服务提供者已经注册在Eureka server上，并且服务名为：“user-service”。需要修改application.yml如下：

```
zuul:
  routes:
    user:
      path: /user/**
      serviceId: user-service
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
此时访问 http://localhost:8080/user/user/list，会转发到 http://localhost:8081/user/list

2.3 Spring Cloud Zuul的默认路由规则

如果服务已注册在Eureka server上，那么zuul会提供默认的路由配置: http://ZUUL_HOST:ZUUL_PORT/serviceId/** 会自动转发到 serviceId对应的为服务上。比如访问 http://localhost:8080/user-service/user/list 会转发到 http://localhost:8081/user/list。
