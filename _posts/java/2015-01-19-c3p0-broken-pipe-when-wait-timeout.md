---
layout: post
title:  "C3P0连接池超时连接失效"
date:   2016-01-19 14:02:25 +0800
categories: java
tags: c3p0
---
使用C3P0做数据库连接池，每天早上第一次打开应用的时候会报数据库链接错误，是因为MYSQL的默认“wait_timeout”为8个小时，如果一个连接空闲时间超过8小时Mysql就会自动断开该连接，而连接池却认为该连接是有效的，因此请求报错。异常中也给出了明确的提示和解决方法：

```
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 63,210,211 milliseconds ago.  The last packet sent successfully to the server was 63,210,212 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
```

<!-- more -->

解决办法有三种：

1、修改MYSQl的配置文件“my.cnf”（Windows中为“my.ini”），延长“wait_timeout”。

2、修改项目中c3p0的配置文件，添加或修改属性“maxIdleTime”，使之小于Mysql的“wait_timeout”值。

{% highlight xml linenos %}
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="driverClass" value="${jdbc.driverClass}" />
	<property name="jdbcUrl" value="${jdbc.url}" />
	<property name="user" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
	<!-- 最大空闲时间 -->
	<property name="maxIdleTime" value="10440"/>
</bean>
</pre>
{% endhighlight %}

3、定时使用连接池中的连接

{% highlight xml linenos %}
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="driverClass" value="${jdbc.driverClass}" />
	<property name="jdbcUrl" value="${jdbc.url}" />
	<property name="user" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
	<!-- 定义所有连接测试都执行的测试语句 -->
	<property name="preferredTestQuery" value="SELECT 1"/>
	<!-- 定义多长时间检查所有连接池中的空闲连接 -->
	<property name="idleConnectionTestPeriod" value="10440"/>
	<!-- 因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable等方法来提升连接测试的性能。 -->
	<property name="testConnectionOnCheckout" value="true"/>
</bean>
{% endhighlight %}

<pre>
严重: Servlet.service() for servlet [dispatcher] in context with path [] threw exception [Request processing failed; nested exception is java.lang.RuntimeException: org.springframework.transaction.CannotCreateTransactionException: Could not open JPA EntityManager for transaction; nested exception is javax.persistence.PersistenceException: org.hibernate.TransactionException: JDBC begin transaction failed: 
org.springframework.transaction.CannotCreateTransactionException: Could not open JPA EntityManager for transaction; nested exception is javax.persistence.PersistenceException: org.hibernate.TransactionException: JDBC begin transaction failed: 
	at org.springframework.orm.jpa.JpaTransactionManager.doBegin(JpaTransactionManager.java:430)
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.getTransaction(AbstractPlatformTransactionManager.java:373)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.createTransactionIfNecessary(TransactionAspectSupport.java:438)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:261)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:95)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:136)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodIntercceptor.invoke(CrudMethodMetadataPostProcessor.java:122)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207)
	at com.sun.proxy.$Proxy51.findOne(Unknown Source)
	at *************************************************
	at com.alibaba.dubbo.common.bytecode.Wrapper1.invokeMethod(Wrapper1.java)
	at com.alibaba.dubbo.rpc.proxy.javassist.JavassistProxyFactory$1.doInvoke(JavassistProxyFactory.java:46)
	at com.alibaba.dubbo.rpc.proxy.AbstractProxyInvoker.invoke(AbstractProxyInvoker.java:72)
	at com.alibaba.dubbo.rpc.protocol.InvokerWrapper.invoke(InvokerWrapper.java:53)
	at com.alibaba.dubbo.rpc.filter.ExceptionFilter.invoke(ExceptionFilter.java:64)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.TimeoutFilter.invoke(TimeoutFilter.java:42)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.monitor.support.MonitorFilter.invoke(MonitorFilter.java:75)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.filter.TraceFilter.invoke(TraceFilter.java:78)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ContextFilter.invoke(ContextFilter.java:60)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.GenericFilter.invoke(GenericFilter.java:112)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.ClassLoaderFilter.invoke(ClassLoaderFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.filter.EchoFilter.invoke(EchoFilter.java:38)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper$1.invoke(ProtocolFilterWrapper.java:91)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.reply(DubboProtocol.java:108)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.handleRequest(HeaderExchangeHandler.java:84)
	at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchangeHandler.received(HeaderExchangeHandler.java:170)
	at com.alibaba.dubbo.remoting.transport.DecodeHandler.received(DecodeHandler.java:52)
	at com.alibaba.dubbo.remoting.transport.dispatcher.ChannelEventRunnable.run(ChannelEventRunnable.java:82)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
Caused by: javax.persistence.PersistenceException: org.hibernate.TransactionException: JDBC begin transaction failed: 
	at org.hibernate.jpa.spi.AbstractEntityManagerImpl.convert(AbstractEntityManagerImpl.java:1763)
	at org.hibernate.jpa.spi.AbstractEntityManagerImpl.convert(AbstractEntityManagerImpl.java:1677)
	at org.hibernate.jpa.spi.AbstractEntityManagerImpl.throwPersistenceException(AbstractEntityManagerImpl.java:1771)
	at org.hibernate.jpa.internal.TransactionImpl.begin(TransactionImpl.java:64)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.beginTransaction(HibernateJpaDialect.java:159)
	at org.springframework.orm.jpa.JpaTransactionManager.doBegin(JpaTransactionManager.java:380)
	... 47 more
Caused by: org.hibernate.TransactionException: JDBC begin transaction failed: 
	at org.hibernate.engine.transaction.internal.jdbc.JdbcTransaction.doBegin(JdbcTransaction.java:76)
	at org.hibernate.engine.transaction.spi.AbstractTransactionImpl.begin(AbstractTransactionImpl.java:162)
	at org.hibernate.internal.SessionImpl.beginTransaction(SessionImpl.java:1435)
	at org.hibernate.jpa.internal.TransactionImpl.begin(TransactionImpl.java:61)
	... 49 more
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 63,210,211 milliseconds ago.  The last packet sent successfully to the server was 63,210,212 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
	at sun.reflect.GeneratedConstructorAccessor66.newInstance(Unknown Source)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:389)
	at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1038)
	at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3609)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2417)
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2582)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2531)
	at com.mysql.jdbc.ConnectionImpl.setAutoCommit(ConnectionImpl.java:4852)
	at com.mchange.v2.c3p0.impl.NewProxyConnection.setAutoCommit(NewProxyConnection.java:881)
	at org.hibernate.engine.transaction.internal.jdbc.JdbcTransaction.doBegin(JdbcTransaction.java:72)
	... 52 more
Caused by: java.net.SocketException: Broken pipe
	at java.net.SocketOutputStream.socketWrite0(Native Method)
	at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:113)
	at java.net.SocketOutputStream.write(SocketOutputStream.java:159)
	at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
	at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
	at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3591)
	... 58 more
] with root cause
</pre>