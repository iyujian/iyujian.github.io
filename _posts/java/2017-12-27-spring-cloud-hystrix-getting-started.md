---
layout: post
title:  "Spring Cloud Hystrix 断路器 - Spring Cloud"
date:   2017-12-27 14:27:51 +0800
categories: java
tags: [Spring Cloud]
---

Hystrix是一个断路器（circuit breaker）。在微服务体系中，通常会有多层的链式的服务调用，如果在调用链中某个环节故障，会导致该请求阻塞，进而导致调用链级联故障，最终影响整个微服务体系的使用，即“雪崩效应”。为了解决这个问题，Netflix推出了Hystrix，当请求某个服务发生故障的频率达到一个阈值时（Hystrix默认是5秒钟内20次故障），会自动断路并返回自定义的静态数据。

## 如何使用Spring Cloud Hystrix？

1. 在pom.xml加入<code>spring-cloud-starter-hystrix</code>依赖。

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

2. 修改启动类，加上注解<code>@EnableCircuitBreaker</code>。

```
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@RestController
@EnableCircuitBreaker
public class HystrixApplication {

	public static void main(String[] args) {
		SpringApplication.run(HystrixApplication.class, args);
	}

	@HystrixCommand(fallbackMethod = "fallbackHello")
	@GetMapping("hello/{name}")
    public String hello(@PathVariable String name) {
	    /*服务提供者的url*/
	    String url = "";
	    return new RestTemplate().getForObject(url, String.class);
    }

    public String fallbackHello(String name) {
	    return "fallback: " + name;
    }
}
```

这样，一个简单的实例就完成了，如果请求url失败会返回fallbackHello方法的执行结果。

（官方文档：http://cloud.spring.io/spring-cloud-static/Edgware.RELEASE/single/spring-cloud.html#_circuit_breaker_hystrix_clients）