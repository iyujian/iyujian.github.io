---
layout: post
title:  "CAS单点登录与Shiro的集成"
date:   2016-03-01 16:17:50 +0800
categories: java
tags: cas shiro
---

1、CAS 与 Shiro 集成不需要修改"web.xml"文件，也不用依赖"cas-client-core"（参考：["CAS单点登录初使用"](cas-first.html "CAS单点登录初使用")），只需要依赖"shiro-cas" 就可以了，因为"shiro-cas"已经依赖了"cas-client-core"。

{% highlight xml %}
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-cas</artifactId>
    <version>1.2.4</version>
</dependency>
{% endhighlight %}

2、然后修改shiro的配置文件，shiro.xml。

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <!-- 会话 Cookie 模板 -->
    <bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
        <constructor-arg value="sid"/>
        <property name="httpOnly" value="true"/>
        <property name="maxAge" value="-1"/>
        <!-- sessionIdCookie：maxAge=-1 表示浏览器关闭时失效此 Cookie； -->
    </bean>
    <bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
        <constructor-arg value="rememberMe"/>
        <property name="httpOnly" value="true"/>
        <property name="maxAge" value="2592000"/>
        <!--30 天，rememberMeCookie即记住我的 Cookie，保存时长 30 天； -->
    </bean>
    <!-- rememberMe 管理器  -->
    <bean id="rememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
        <!-- cipherKey 是加密 rememberMe Cookie 的密钥；默认 AES 算法 -->
        <property name="cipherKey" value="#{T(org.apache.shiro.codec.Base64).decode('4AvVhmFLUs0KTA3Kprsdag==' )}"/>
        <property name="cookie" ref="rememberMeCookie"/>
    </bean>

    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="filters">
            <util:map>
                <entry key="cas" value-ref="casFilter"/>
            </util:map>
        </property>
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="http://localhost:8080/login?service=http://localhost:8081/welcome" />
        <property name="unauthorizedUrl" value="/unauthorized"/>
        <property name="filterChainDefinitions">
            <value>
                /welcome = cas
                /unauthorized = anon
                /logout = logout
                /common/** = anon
                /static/** = anon
                /** = authc
            </value>
        </property>
    </bean>
    <bean id="casSubjectFactory" class="org.apache.shiro.cas.CasSubjectFactory"/>
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="realm"/>
        <property name="cacheManager" ref="cacheManager"/>
        <!-- 设置 securityManager 安全管理器的 rememberMeManager -->
        <property name="rememberMeManager" ref="rememberMeManager"/>
        <property name="subjectFactory" ref="casSubjectFactory"/>
    </bean>
    
    <bean id="realm" class="com.ces.common.erp.security.realm.MyRealm">
        <property name="cachingEnabled" value="true"/>
        <property name="authenticationCachingEnabled" value="true"/>
        <property name="authenticationCacheName" value="authenticationCache"/>
        <property name="authorizationCachingEnabled" value="true"/>
        <property name="authorizationCacheName" value="authorizationCache"/>
        <!--CAS Server 地址-->
        <property name="casServerUrlPrefix" value="http://localhost:8080"/>
        <!--当前应用CAS服务URL，即用于接收并处理登录成功后的Ticket-->
        <property name="casService" value="http://localhost:8081/welcome"/>
    </bean>

    <bean id="casFilter" class="org.apache.shiro.cas.CasFilter">
        <property name="failureUrl" value="/unauthorized"/>
    </bean>

    <!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"/>

</beans>
{% endhighlight %}

["CAS单点登录初使用"](cas-first.html "CAS单点登录初使用")

["CAS单点登录密码加盐的认证"](cas-username-password-salt-authentication-handler.html "CAS单点登录密码加盐的认证")

["CAS+Shiro实现单点退出"](cas-shiro-logout.html "CAS+Shiro实现单点退出")

["基于Redis的CAS服务端集群"](cluster-of-cas-server-by-redis.html "基于Redis的CAS服务端集群")