---
layout: post
title:  "Spring Cloud Ribbon 客户端侧负载均衡器 - Spring Cloud"
date:   2017-12-05 13:44:33 +0800
categories: java
tags: [Spring Cloud]
---

Ribbon是一个客户端侧负载均衡器（Client Side Load Balancer）。每个Ribbon客户端都会从服务注册中心（Eureka）获取服务清单并缓存在客户端，同时通过心跳机制去维护服务清单，然后执行负载均衡策略去调用不同服务器上的服务，谓之客户端侧负载均衡，因为这个负载均衡是发生在客户端侧的，而服务端侧负载均衡，例如Nginx，负载均衡是发生在服务端的，客户端发起请求，Nginx负责维护服务清单，并执行负载均衡策略。

> Ribbon用Eureka作为注册中心，ribbon需要作为Eureka client注册在Eureka server上。

1. 添加Ribbon和Eurekak客户端依赖。

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
{% endhighlight %}

2. application.yml

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

3. Spring boot 启动类，“service-provider”为服务提供者的应用名称，即服务提供者的“spring.application.name”。

{% highlight java %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
@RestController
public class RibbonApplication {

	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}

    @Autowired
    private RestTemplateBuilder restTemplateBuilder;

	@Bean
    @LoadBalanced
    RestTemplate restTemplate() {
	    return restTemplateBuilder.build();
    }

    @GetMapping
    public String hello() {
	    return restTemplate().getForObject("http://service-provider/hello", String.class);
    }
}
{% endhighlight %}