---
layout: post
title:  "Apache Shiro 自定义 Realm 实现登录及授权"
date:   2015-11-06 09:41:55 +0800
categories: java
tags: shiro
---
在上一篇文章讲了Shiro自带的JdbcRealm实现的登录和授权，但是这种方式不能在principal中传入对象，所以我们可能需要自定义Realm的实现。

1、自定义MyRealm继承AuthorizingRealm
{% highlight java linenos %}
import com.***.User;
import com.***.userService;
import com.***.roleService;
import com.***.resourceService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.List;

public class MyRealm extends AuthorizingRealm {

    @Autowired
    private UserService userService;

    @Autowired
    private RoleService roleService;

    @Autowired
    private ResourceService resourceService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

        User user = (User) principals.getPrimaryPrincipal();

        List<String> roles = roleService.getRoles(user.getId());

        List<String> resources = resourceService.getResources(user.getId());

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        info.addRoles(roles);

        info.addStringPermissions(resources);

        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        String username = usernamePasswordToken.getUsername();

        User user = userService.getUserByUsername(username);

        if (null != user) {
            //此处传入的第一个参数为user对象，那么我们在页面可以通过这样的方式来获取用户名：<shiro:principal property="username"/>
            return new SimpleAuthenticationInfo(user, user.getPassword(), ByteSource.Util.bytes(user.getSalt()), getName());
        }

        return null;
    }
}
{% endhighlight %}
2、修改shiro.xml
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
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

    <bean id="realm" class="com.ces.common.core.security.realm.MyRealm">
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
{% endhighlight  %}

3、其他的配置参考上一篇：
["Apache Shiro JdbcRealm 实现登录及授权"](shiro-jdbcrealm-of-authorizing-realm.html "Apache Shiro JdbcRealm 实现登录及授权")
