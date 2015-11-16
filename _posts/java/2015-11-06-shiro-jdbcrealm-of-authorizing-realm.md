---
layout: post
title:  "Apache Shiro JdbcRealm 实现登录及授权"
date:   2015-11-06 09:15:38 +0800
categories: java
tags: shiro
---
以下是使用shiro自身的JdbcRealm实现的登录及授权功能：

<!-- more -->

1、pom文件中添加依赖

{% highlight xml linenos %}
<dependency>
	<groupId>org.apache.shiro</groupId>

	<artifactId>shiro-core</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-web</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-ehcache</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring</artifactId>
</dependency>
{% endhighlight %}

2、web.xml中添加shiroFilter
{% highlight xml linenos %}
<!-- Shiro Filter -->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>ERROR</dispatcher>
</filter-mapping>
{% endhighlight %}

3、为shiro添加spring的配置文件，shiro.xml，然后在主配置文件中import。
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

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
        <property name="securityManager" ref="securityManager"/>
        <!-- 缺省为： /login.jsp <property name="loginUrl" value="/login.jsp" /> -->
        <!-- 缺省为：/ <property name="successUrl" value="/" /> -->
        <property name="unauthorizedUrl" value="/403.jsp"/>
        <property name="filterChainDefinitions">
            <value>
                /login.jsp = authc
                /logout = logout
                /**.* = anon
                /** = authc
            </value>
        </property>
    </bean>

    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="realm"/>
        <property name="cacheManager" ref="cacheManager"/>
        <!-- 设置 securityManager 安全管理器的 rememberMeManager -->
        <property name="rememberMeManager" ref="rememberMeManager"/>
    </bean>

    <bean id="realm" class="org.apache.shiro.realm.jdbc.JdbcRealm">
        <property name="dataSource" ref="dataSource"/>
        <!-- 登录时查询的SQL -->
        <property name="authenticationQuery" value="select password, salt from user where username = ?"/>
        <!-- 角色授权查询的SQL：根据用户名查询角色 -->
        <property name="userRolesQuery" value="select role_code from ***** where username = ?"/>
        <!-- 权限资源授权查询的SQL：根据角色CODE查询资源 -->
        <property name="permissionsQuery" value="select resource_code from ***** where role_code = ?"/>
        <!-- permissionsLookupEnabled：默认false。False时不会使用permissionsQuery的SQL去查询权限资源 -->
        <property name="permissionsLookupEnabled" value=""/>
        <property name="saltStyle" value="COLUMN"/>
        <property name="credentialsMatcher" ref="hashedCredentialsMatcher"/>
    </bean>
    <bean id="hashedCredentialsMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
        <property name="hashAlgorithmName" value="MD5"/>
        <property name="storedCredentialsHexEncoded" value="true"/>
        <property name="hashIterations" value="1024"/>
    </bean>

    <!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"/>

</beans>
{% endhighlight %}

在applicationContext.xml中导入shiro.xml文件

{% highlight xml %}
<import resource="classpath:shiro/shiro.xml" />
{% endhighlight %}

以上的实现为密码加盐的实现，因此在生成密码的时候要注意使用下面的密码生成方式：并将方法返回的密码和salt存入数据库。

{% highlight java linenos %}
import org.apache.shiro.crypto.RandomNumberGenerator;
import org.apache.shiro.crypto.SecureRandomNumberGenerator;
import org.apache.shiro.crypto.hash.Md5Hash;

import java.util.HashMap;
import java.util.Map;

public class PasswordUtil {

    public static Map<String, String> encrypt(String password) {

        RandomNumberGenerator rng = new SecureRandomNumberGenerator();
        String salt = rng.nextBytes().toHex();
        //此处的1024对应shiro.xml中的hashIterations
        password = new Md5Hash(password, salt, 1024).toHex();

        Map<String, String> map = new HashMap<String, String>();

        map.put("password", password);
        map.put("salt", String.valueOf(salt));

        return map;
    }

}
{% endhighlight %}

4、实现shiro的注解授权

需要在springmvc的配置文件中加入配置：
{% highlight xml linenos %}
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor"/>
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	<property name="securityManager" ref="securityManager"/>
</bean>
{% endhighlight %}

5、在方法上加上注解即可实现授权，
{% highlight java linenos %}
@RequiresPermissions("user:add")
@RequestMapping("add")
public String add() {
    return "user/add";
}

@RequiresRoles("admin")
@RequestMapping("list")
public String list() {
    return "user/list";
}
{% endhighlight %}