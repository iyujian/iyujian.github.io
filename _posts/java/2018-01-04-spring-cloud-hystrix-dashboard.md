---
layout: post
title:  "Spring Cloud Hystrix Dashboard - Spring Cloud"
date:   2018-01-04 13:08:52 +0800
categories: java
tags: [Spring Cloud]
---

["Spring Cloud Hystrix 断路器"](spring-cloud-hystrix-getting-started.html "Spring Cloud Hystrix 断路器")

Hystrix可以收集每个断路器（以<code>HystrixCommand</code>注解的）的运行状态， 由 Hystrix Dashboard 以界面化的方式进行展示。

1. 需要的依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

2. 加上<code>@EnableHystrixDashboard</code>注解

3. 启动工程后访问：http://localhost:8080/hystrix，输入http://localhost:8080/hystrix.stream，点击按钮“Monitor Stream”，即可进入Hystrix Dashboard，此时进去面板显示“Loading ...”，需要在新的浏览器窗口中访问由<code>@HystrixCommand</code>注解的请求（http://localhost:8080/hello/world），即可。