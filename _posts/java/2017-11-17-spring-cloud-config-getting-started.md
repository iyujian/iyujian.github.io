---
layout: post
title:  "Spring Cloud Config 入门 - Spring Cloud"
date:   2016-07-13 10:58:49 +0800
categories: java
tags: [Spring Cloud]
---

什么是Spring cloud config? 官方文档地址： http://projects.spring.io/spring-cloud/spring-cloud.html#_spring_cloud_config

> 1、Spring cloud config repo

创建git仓库，用于集中存放配置文件，地址为：https://github.com/spring-cloud-samples/config-repo.git

创建配置文件：foo-development.properties，内容为：

```
bar: config
foo: from foo development
```

> 2、Spring cloud config server

创建一个Spring boot应用，pom.xml添加依赖：

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
{% endhighlight %}

启动类加上“@EnableConfigServer”注解开启服务。

创建bootstrap.yml，加入config repo的配置：

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo.git
          username: ***
          password: ***
```

启动应用，输入地址： http://localhost:8080/foo/development，输入结果：

```
{"name":"foo","profiles":["development"],"label":null,"version":"a8ac18f8579d63c2bec3a6d961d776e268efaaeb","state":null,"propertySources":[{"name":"https://github.com/spring-cloud-samples/config-repo.git/foo-development.properties","source":{"bar":"spam","foo":"from foo development"}}]}
```

此时服务端已经配置成功。

> 3、Spring cloud config client

创建Spring boot应用，pom.xml加入依赖：

{% highlight xml %}
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
{% endhighlight %}

修改application.yml，将应用端口改为8081

创建bootstrap.yml，加入配置：

```
spring:
  cloud:
    config:
      name: foo
      profile: development
      label: master
      uri: http://localhost:8080
```

修改Spring boot启动类

{% highlight java %}
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class ConfigClientApplication {

    @Value("${bar:World!}")
    private String bar;

    @GetMapping("/")
    String hello() {
        return "Hello " + bar + "!";
    }

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}
}
{% endhighlight %}

运行localhost:8081，输入： Hello config!
