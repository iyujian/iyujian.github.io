---
layout: post
title:  "CAS单点登录密码加盐的认证"
date:   2016-03-01 17:05:36 +0800
categories: java
tags: cas shiro
---

为了安全，我们会对系统中的密码进行特殊方法加密，比如加入密码盐，并用MD5迭代N次，这个时候我们就需要在CAS服务端自定义认证处理类。

1、在CAS服务端新建类"MyAuthenticationHandler"，继承自"AbstractJdbcUsernamePasswordAuthenticationHandler"。

{% highlight java %}
package com.***.authentication.handler;

import org.apache.shiro.crypto.hash.Md5Hash;
import org.jasig.cas.adaptors.jdbc.AbstractJdbcUsernamePasswordAuthenticationHandler;
import org.jasig.cas.authentication.HandlerResult;
import org.jasig.cas.authentication.PreventedException;
import org.jasig.cas.authentication.UsernamePasswordCredential;
import org.springframework.dao.DataAccessException;
import org.springframework.dao.IncorrectResultSizeDataAccessException;

import javax.security.auth.login.AccountNotFoundException;
import javax.security.auth.login.FailedLoginException;
import javax.validation.constraints.NotNull;
import java.security.GeneralSecurityException;
import java.util.Map;

public class MyAuthenticationHandler extends AbstractJdbcUsernamePasswordAuthenticationHandler {

    @NotNull
    private String sql;

    /**
     * {@inheritDoc}
     */
    @Override
    protected final HandlerResult authenticateUsernamePasswordInternal(final UsernamePasswordCredential credential)
            throws GeneralSecurityException, PreventedException {

        final String username = credential.getUsername();
        try {
            final Map<String, Object> values = getJdbcTemplate().queryForMap(this.sql, username);
            String dbPassword = (String) values.get("password");
            String salt = (String) values.get("salt");
            Integer status = (Integer) values.get("status");
            if(!status.equals(1)) { //判断帐号是否被冻结
                throw new FailedLoginException("This user is disabled.");
            }

            //密码加盐并迭代1024次
            String encryptedPassword = new Md5Hash(credential.getPassword(), salt, 1024).toHex();
            if (!dbPassword.equals(encryptedPassword)) {
                throw new FailedLoginException("Password does not match value on record.");
            }
        } catch (final IncorrectResultSizeDataAccessException e) {
            if (e.getActualSize() == 0) {
                throw new AccountNotFoundException(username + " not found with SQL query");
            } else {
                throw new FailedLoginException("Multiple records found for " + username);
            }
        } catch (final DataAccessException e) {
            e.printStackTrace();
            throw new PreventedException("SQL exception while executing query for " + username, e);
        }
        return createHandlerResult(credential, this.principalFactory.createPrincipal(username), null);
    }

    /**
     * @param sql The sql to set.
     */
    public void setSql(final String sql) {
        this.sql = sql;
    }
}
{% endhighlight %}

2、修改CAS服务端的"deployerConfigContext.xml"文件。

{% highlight xml %}
<bean id="primaryAuthenticationHandler" class="com.***.authentication.handler.MyAuthenticationHandler">
    <property name="dataSource" ref="dataSource"/>
    <property name="sql" value="SELECT password, salt, status FROM table_name WHERE username=?"/>
</bean>

<!-- 数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driverClassName}" />
    <property name="jdbcUrl" value="${jdbc.url}" />
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="maxIdleTime" value="14400"/>
</bean>
{% endhighlight %}

["CAS单点登录初使用"](cas-first.html "CAS单点登录初使用")

["CAS单点登录与Shiro的集成"](integration-of-cas-and-shiro.html "CAS单点登录与Shiro的集成")

["CAS+Shiro实现单点退出"](cas-shiro-logout.html "CAS+Shiro实现单点退出")