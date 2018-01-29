---
layout: post
title:  "Spring Cloud Security Oauth2 Client - Spring Cloud"
date:   2018-01-29 14:05:46 +0800
categories: java
tags: [Spring Cloud]
---

利用Spring Cloud Security Oauth2如何对接第三方的认证和授权服务？以集成Github SSO为例：

1、注册github帐号并申请一个Oauth应用。

打开地址：https://github.com/settings/developers，点击按钮“Register a new application”，按要求填写信息即可。申请成功后会有“Client ID”和“Client Secret”。

注意：申请时填写的“Authorization callback URL”（http://localhost:8080/login）需要与<code>security.oauth2.sso.loginPath</code>（默认为: /login）对应。

2、添加<code>spring-cloud-starter-oauth2</code>依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```

3、修改 application.yml 配置文件

```yaml
security:
  oauth2:
    client:
      clientId: bd1c0a783ccdd1c9b9e4
      clientSecret: 1a9030fbca47a5b2c28e92f19050bb77824b5ad1
      accessTokenUri: https://github.com/login/oauth/access_token
      userAuthorizationUri: https://github.com/login/oauth/authorize
      clientAuthenticationScheme: form
    resource:
      userInfoUri: https://api.github.com/user
      preferTokenInfo: false
```

4、Spring boot启动类：
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.security.oauth2.client.EnableOAuth2Sso;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.security.Principal;

@SpringBootApplication
@RestController
@EnableOAuth2Sso
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class, args);
    }

    @GetMapping("hello/{name}")
    public Object hello(@PathVariable String name) {
        return "Hello, " + name + ".";
    }

    @GetMapping("/user")
    public Principal user(Principal user) {
        return user;
    }

}
```

<code>@EnableOAuth2Sso</code>用于开启SSO（Single Sign On，单点登录）服务，启动工程后，访问工程内资源即会先跳转到github进行认证。

官方示例源码：https://github.com/spring-cloud-samples/sso