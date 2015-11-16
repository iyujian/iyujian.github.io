---
layout: post
title:  "一个Dubbo的小例子"
date:   2015-11-10 12:39:37 +0800
categories: java
tags: dubbo
---

提供者（Provider），消费者（Consumer）

1、服务提供者（Provider）部分

1) pom文件中添加zookeeper和dubbo的依赖，提供者和消费者都需要加上。

{% highlight xml linenos %}
<!-- Zookeeper -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.0-alpha</version>
</dependency>

<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.5</version>
</dependency>

<!-- Dubbo -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.3</version>
</dependency>
{% endhighlight %}

2) 提供者接口的定制(该接口需单独打包，在服务提供方和消费方共享)：

{% highlight java linenos %}
public interface HelloWorldService {
    public String sayHello(String str);
}
{% endhighlight %}

3) 提供者接口的实现(对服务消费方隐藏实现)：

{% highlight java linenos %}
public class HelloWorldServiceImpl implements HelloWorldService {

    @Override
    public String sayHello(String str) {
        return "hello, " + str + "!";
    }

}
{% endhighlight %}

4) 提供者的spring配置文件部分（声明暴露服务）：

{% highlight xml linenos %}

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.***.HelloWorldService" ref="helloWorldService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="helloWorldService" class="com.***.impl.HelloWorldServiceImpl" />
 
</beans>
{% endhighlight %}

2、服务消费者（Consumer）部分

1) pom中加入对服务提供者接口的依赖
{% highlight xml linenos %}
<dependency>
    <groupId>com.group</groupId>
    <artifactId>dubbo-demo-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
{% endhighlight %}

2) 通过Spring配置引用远程服务：
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用zookeeper注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="helloWorldService" interface="com.***.HelloWorldService" />
 
</beans>
{% endhighlight %}
3) 加载Spring配置，并调用远程服务：(也可以使用IoC注入)
{% highlight java linenos %}
package com.dubbo.helloworld;

import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.***.HelloWorldService;

public class Consumer {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
                new String[]{"applicationContext.xml"});

        HelloWorldService service = (HelloWorldService) context.getBean("helloWorldService");

        System.out.println(service.sayHello("world"));
    }
}
{% endhighlight %}

dubbo用户指南：http://dubbo.io/User+Guide-zh.htm

["基于ZooKeeper的Dubbo注册中心安装(CentOS)"](zookeeper-and-dubbo-admin-installation.html "基于ZooKeeper的Dubbo注册中心安装(CentOS)")