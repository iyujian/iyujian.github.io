---
layout: post
title:  "基于Redis的CAS服务端集群"
date:   2016-03-31 17:23:57 +0800
categories: java
tags: cas redis
---
为了保证生产环境CAS(Central Authentication Service)认证服务的高可用，防止出现单点故障，我们需要对CAS Server进行集群部署。

CAS的Ticket默认是以Map的方式存储在JVM内存中的，多个tomcat之间无法共享，因此我们可以使用MemCached或者Redis来存储Ticket。MemCache的方式官方已经提供了解决方案，我们这里使用的是Redis，具体实现可以参考CAS的模块：cas-server-integration-memcached。

1、pom.xml文件中加入依赖：

{% highlight xml %}
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.7.2</version>
</dependency>
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-redis</artifactId>
  <version>1.6.0.RELEASE</version>
</dependency>
{% endhighlight %}

2、参考：org.jasig.cas.ticket.registry.MemCacheTicketRegistry，重写Ticket注册类。

{% highlight java %}
package com.***.cas.ticket.registry;

import org.jasig.cas.ticket.ServiceTicket;
import org.jasig.cas.ticket.Ticket;
import org.jasig.cas.ticket.TicketGrantingTicket;
import org.jasig.cas.ticket.registry.AbstractDistributedTicketRegistry;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.data.redis.core.RedisTemplate;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import java.util.Collection;
import java.util.concurrent.TimeUnit;

public final class RedisTicketRegistry extends AbstractDistributedTicketRegistry implements DisposableBean {

    /** Memcached client. */
    @NotNull
    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * TGT cache entry timeout in seconds.
     */
    @Min(0)
    private final int tgtTimeout;

    /**
     * ST cache entry timeout in seconds.
     */
    @Min(0)
    private final int stTimeout;

    public RedisTicketRegistry(RedisTemplate<String, Object> redisTemplate, int tgtTimeout, int stTimeout) {
        this.redisTemplate = redisTemplate;
        this.tgtTimeout = tgtTimeout;
        this.stTimeout = stTimeout;
    }

    @Override
    public void addTicket(Ticket ticket) {
        logger.debug("Adding ticket {}", ticket);
        try {
            this.redisTemplate.opsForValue().set(ticket.getId(),ticket, getTimeout(ticket), TimeUnit.SECONDS);
        } catch (Exception e) {
            logger.error("Failed adding {}", ticket, e);
        }

    }

    @Override
    public Ticket getTicket(String ticketId) {
        try {
            final Ticket t = (Ticket) this.redisTemplate.opsForValue().get(ticketId);
            if (t != null) {
                return getProxiedTicketInstance(t);
            }
        } catch (final Exception e) {
            logger.error("Failed fetching {} ", ticketId, e);
        }
        return null;
    }

    @Override
    public boolean deleteTicket(String ticketId) {
        if (ticketId == null) {
            return false;
        }

        final Ticket ticket = getTicket(ticketId);
        if (ticket == null) {
            return false;
        }

        logger.debug("Deleting ticket {}", ticketId);
        try {
            this.redisTemplate.delete(ticketId);
        } catch (final Exception e) {
            logger.error("Failed deleting {}", ticketId, e);
        }
        return false;
    }

    @Override
    protected void updateTicket(Ticket ticket) {
        logger.debug("Updating ticket {}", ticket);
        try {
            if(this.redisTemplate.hasKey(ticket.getId())) {
                this.redisTemplate.opsForValue().set(ticket.getId(), ticket, getTimeout(ticket), TimeUnit.SECONDS);
            }
        } catch (final Exception e) {
            logger.error("Failed updating {}", ticket, e);
        }
    }

    @Override
    public Collection<Ticket> getTickets() {
        throw new UnsupportedOperationException("GetTickets not supported.");
    }

    @Override
    protected boolean needsCallback() {
        return true;
    }

    @Override
    public void destroy() throws Exception {
        
    }

    /**
     * Gets the timeout value for the ticket.
     *
     * @param t the t
     * @return the timeout
     */
    private int getTimeout(final Ticket t) {
        if (t instanceof TicketGrantingTicket) {
            return this.tgtTimeout;
        } else if (t instanceof ServiceTicket) {
            return this.stTimeout;
        }
        throw new IllegalArgumentException("Invalid ticket type");
    }
}
{% endhighlight %}

3、修改“ticketRegistry.xml”，先删除文件中所有的bean定义，包括ticketRegistry、ticketRegistryCleaner、jobDetailTicketRegistryCleaner和triggerJobDetailTicketRegistryCleaner，然后添加下面的代码片段：

{% highlight xml %}
<bean id="ticketRegistry" class="com.***.cas.ticket.registry.RedisTicketRegistry">
    <constructor-arg index="0" ref="redisTemplate" />
    <constructor-arg index="1" value="1800" />
    <constructor-arg index="2" value="10" />
</bean>
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxIdle" value="200" />
    <property name="testOnBorrow" value="true" />
</bean>
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <property name="hostName" value="${redis.host}"/>
    <property name="port" value="6379"/>
    <property name="poolConfig" ref="jedisPoolConfig"/>
</bean>
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory"/>
</bean>
{% endhighlight %}

到此，基于Redis的CAS Server集群配置已经完成。


["CAS单点登录初使用"](cas-first.html "CAS单点登录初使用")

["CAS单点登录与Shiro的集成"](integration-of-cas-and-shiro.html "CAS单点登录与Shiro的集成")

["CAS单点登录密码加盐的认证"](cas-username-password-salt-authentication-handler.html "CAS单点登录密码加盐的认证")

["CAS+Shiro实现单点退出"](cas-shiro-logout.html "CAS+Shiro实现单点退出")