---
layout: post
title:  "Spring Cloud Feign对Hystrix断路器的配置 - Spring Cloud"
date:   2017-12-28 14:34:11 +0800
categories: java
tags: [Spring Cloud]
---

前面分别讲过了 ["Spring Cloud Feign 声明式REST客户端"](spring-cloud-feign-getting-started.html "Spring Cloud Feign 声明式REST客户端") 和 ["Spring Cloud Hystrix 断路器"](spring-cloud-hystrix-getting-started.html "Spring Cloud Hystrix 断路器")，其实在Feign中已经集成了对Hystrix的配置。

基于之前的Feign工程，做如下改造：

1. application.yml添加配置：

```xml
feign:
  hystrix:
    enabled: true
```

2. 修改Feign client

```java
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "${provider.hostname}", fallback = HelloClient.HelloClientFallback.class)
public interface HelloClient {

    @GetMapping("hello/{name}")
    String hello(@PathVariable(value = "name") String name);

    @Component
    class HelloClientFallback implements HelloClient {
        @Override
        public String hello(String name) {
            return "Fallback: " + name;
        }
    }

}
```

