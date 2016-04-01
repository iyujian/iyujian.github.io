---
layout: post
title:  "CAS+Shiro实现单点退出"
date:   2016-03-11 13:28:06 +0800
categories: java
tags: cas shiro
---
CAS+Shiro实现了统一认证，但是在做登出操作的时候子系统并没有退出，依然可以访问。

1、配置“web.xml”，在其他filter之前添加：

{% highlight xml %}
<!-- 单点退出 begin -->
<listener>
    <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>
</listener>
<filter>
    <filter-name>singleSignOutFilter</filter-name>
    <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>singleSignOutFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
<!-- 单点退出 end -->
{% endhighlight %}

2、在shiro的配置文件中，添加：

{% highlight xml %}
<bean id="logout" class="org.apache.shiro.web.filter.authc.LogoutFilter">
    <property name="redirectUrl" value="${cas.url}logout?service=${host.url}"/>
</bean>
{% endhighlight %}

3、修改shiro配置文件中的“shiroFilter”，将Shiro默认的logoutFilter替换为上面定义的:

{% highlight xml %}
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="filters">
        <util:map>
        	…………
            <entry key="logout" value-ref="logout" />
        </util:map>
    </property>
    …………
</bean>
{% endhighlight %}

4、修改CAS服务器端的“cas-servlet.xml”文件。将“p:followServiceRedirects="${cas.logout.followServiceRedirects:false}"”改为true，以便退出后可以重定向到登录页。

{% highlight xml %}
<bean id="logoutAction" class="org.jasig.cas.web.flow.LogoutAction"
    p:servicesManager-ref="servicesManager"
    p:followServiceRedirects="${cas.logout.followServiceRedirects:true}"/>
{% endhighlight %}

["CAS单点登录初使用"](cas-first.html "CAS单点登录初使用")

["CAS单点登录与Shiro的集成"](integration-of-cas-and-shiro.html "CAS单点登录与Shiro的集成")

["CAS单点登录密码加盐的认证"](cas-username-password-salt-authentication-handler.html "CAS单点登录密码加盐的认证")

["基于Redis的CAS服务端集群"](cluster-of-cas-server-by-redis.html "基于Redis的CAS服务端集群")