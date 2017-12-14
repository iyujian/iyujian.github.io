---
layout: post
title:  "Spring Cloud Feign 声明式REST客户端 - Spring Cloud"
date:   2017-12-14 11:15:31 +0800
categories: java
tags: [Spring Cloud]
---

Feign是一个声明式Web服务客户端，使得调用Rest服务更简单。使用Feign只需要写一个接口加上一些注解就可以。下面主要介绍如何和Eureka一起使用。

> 1、假设我们已经启动了Eureka server(http://localhost:8671) 和 服务提供者（提供了Rest服务：http://localhost:8888/hello/{name}），并且服务提供者已经在Eureka server上注册。

> 2、新建spring boot工程，加入Feign和Eureka客户端依赖。

{% highlight xml %}
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
{% endhighlight %}

> 3、 在application.yml中加入Eureka client配置和服务提供者注册在Eureka server上的服务名（“spring.application.name”）。

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
provider:
  hostname: service-provider
```

> 4、 新建Feign Client接口文件。

{% highlight java %}
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient("${provider.hostname}")
public interface HelloClient {

    @RequestMapping(method = RequestMethod.GET, value = "hello/{name}")
    String hello(@PathVariable(value = "name") String name);

}
{% endhighlight %}

> 5、 启动类加上注解：@EnableFeignClients、@EnableEurekaClient等。

{% highlight java %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@EnableEurekaClient
@SpringBootApplication
@RestController
@EnableFeignClients
public class FeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }

    @Autowired
    private HelloClient helloClient;

    @GetMapping(value = "hello")
    public String hello() {
        return helloClient.hello("feign");
    }
}
{% endhighlight %}

> 6、 启动工程，访问 http://localhost:8080/hello。

> 7、遇到的问题：

```
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'feignApplication': Unsatisfied dependency expressed through field 'helloClient'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'com.***.HelloClient': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: PathVariable annotation was empty on param 0.
```

上面这个错误是因为在@FeignClient注解的接口中用到的参数注解“@PathVariable”必须设置"value"值。