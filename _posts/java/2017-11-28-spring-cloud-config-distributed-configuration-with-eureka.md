---
layout: post
title:  "Spring Cloud Config/Eureka 构建分布式配置中心 - Spring Cloud"
date:   2017-11-28 16:31:41 +0800
categories: java
tags: [Spring Cloud]
---

前文提到了["Spring cloud config"](spring-cloud-config-getting-started.html "Spring cloud config如何使用")和["Eureka"](spring-cloud-eureka-getting-started.html "Eureka如何使用")的使用。此时只需要针对config server和config client做一些改造，通过Eureka来实现集群服务。

> 1、改造Config Server，将其注册在Eureka server上。pom.xml加入依赖eureka client依赖

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
{% endhighlight %}

修改application.yml，设置应用名并指定注册中心。

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
spring:
  application:
    name: config-server
```

在Spring boot启动类添加注解“@EnableDiscoveryClient”或者“@EnableEurekaClient”。

> 2、改造Config client。 同上，将也注册在eureka上，同时在bootstrap.yml配置config-server和eureka信息：

```
spring:
  cloud:
    config:
      name: foo
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

启动类如下：

{% highlight java %}
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
@EnableEurekaClient
public class ConfigClientApplication {

    @Value("${foo}")
    private String foo;

    @GetMapping("/")
    public String hello() {
        return foo;
    }

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}
}
{% endhighlight %}

> 3、依次启动eureka-server，config-server，config-client，访问config-client，看是否已经获取到config-server提供的配置信息。

过程中遇到一个异常：“java.lang.IllegalStateException: No instances found of configserver”，需要把此前config client的eureka配置和config server配置都写在bootstrap.yml文件中，而不是application.yml。