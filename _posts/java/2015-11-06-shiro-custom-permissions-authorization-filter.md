---
layout: post
title:  "Apache Shiro 动态授权"
date:   2015-11-6 12:48:15 +0800
categories: java
tags: shiro
---
shiro的权限控制可以配置在shiroFilter的filterChainDefinitions中：

<!-- more -->

{% highlight xml linenos %}
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <!-- 此处省略其他配置信息 -->
    <property name="filterChainDefinitions">
        <value>
            /login.jsp = authc
            /logout = logout
            /**.* = anon
            /user/add = perms[user:add]
            /** = authc
        </value>
    </property>
</bean>
{% endhighlight %}

但是一般我们的系统中的权限都是动态的，角色和权限信息都存储于数据库中，此时就需要我们去继承PermissionsAuthorizationFilter和RolesAuthorizationFilter实现基于权限资源的权限控制和基于角色的权限控制。

1、继承PermissionsAuthorizationFilter，重写isAccessAllowed方法，角色授权与此类似。
{% highlight java linenos %}
import com.***.ResourceEntity;
import com.***.ResourceManager;
import org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.util.List;

@Component
public class MyPermissionsAuthorizationFilter extends PermissionsAuthorizationFilter {

    @Autowired
    private ResourceManager resourceManager;

    @Override
    public boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws IOException {
        return super.isAccessAllowed(request, response, getMappedValue(request));
    }

    /**
     * 得到mappedValue，相当于perms[user:add]中的“user:add”
     * @param path
     * @return
     */
    public String[] getMappedValue(ServletRequest request) {
        HttpServletRequest req = (HttpServletRequest) request;
        String path = req.getServletPath();
        String code = getCodesByPath(path);
        if(null == code) {
            return null;
        }
        return new String[]{code};
    }

    /**
     * 根据访问路径获取权限代码
     * @param path
     * @return
     */
    public String getCodesByPath(String path) {
        List<ResourceEntity> list = resourceManager.findBy("url", path);
        if(null != list && list.size()>0) {
            return list.get(0).getCode();
        }
        return null;
    }
}
{% endhighlight %}
2、配置shiro.xml文件
{% highlight xml linenos %}
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="filters">
        <util:map>
            <entry key="perms" value-ref="myPermissionsAuthorizationFilter"></entry>
        </util:map>
    </property>
    <property name="securityManager" ref="securityManager"/>
    <!--<property name="loginUrl" value="/login" /-->
    <!-- 缺省为：/ <property name="successUrl" value="/" /> -->
    <property name="unauthorizedUrl" value="/403.jsp"/>
    <property name="filterChainDefinitions">
        <value>
            /login.jsp = authc
            /logout = logout
            /**.* = anon
            /** = authc,perms
        </value>
    </property>
</bean>
{% endhighlight %}
4、继承RolesAuthorizationFilter实现基于角色的权限控制与上面差不多，不再赘述。