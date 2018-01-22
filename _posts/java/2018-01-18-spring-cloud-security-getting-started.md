---
layout: post
title:  "Spring Cloud Security 权限控制 - Spring Cloud"
date:   2018-01-18 10:23:29 +0800
categories: java
tags: [Spring Cloud]
---

Spring Security 为基于Java EE的企业软件应用提供了全面的安全服务，而Spring cloud security可以极大的简化我们的配置，一个最简单的工程只需要在Spring boot工程中依赖<code>spring-cloud-starter-security</code>即可。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

1、默认用户名为user，密码在启动日志中查找如下内容：

```text
Using default security password: 05be6203-6bfc-4f80-8b26-187af9c8e21d
```

2、如果要自定义用户名和密码，只需修改配置文件application.yml

```yaml
security:
  user:
    name: admin
    password: 12345678
```

3、使用数据库（MYSQL）中用户名和密码进行验证：

3.1、加入需要的依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

3.2、application.yml中配置datasource

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/dbname?characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
```
3.3、添加认证配置类

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

import javax.sql.DataSource;

@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private DataSource dataSource;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .usersByUsernameQuery("select username,password,enabled from users where username = ?")
                .authoritiesByUsernameQuery("select username,authority from authorities where username = ?");
    }
}
```