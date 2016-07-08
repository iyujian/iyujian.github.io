---
layout: post
title:  "如何在Spring启动后执行一些操作"
date:   2015-11-11 13:34:44 +0800
categories: java
tags: spring
---
某些时候，我们在项目启动后就需要执行一些操作，这个时候我们就可以通过实现ApplicationListener接口，实现他的onApplicationEvent方法来做到。

但是直接做的话会有个问题，就是在spring启动后onApplicationEvent方法会被执行两次。

这是因为此时存在两个ApplicationContext：

一个是Root WebApplicationContext；

另一个是SpringMVC的ApplicationContext，叫做：WebApplicationContext for namespace 'dispatcher-servlet'（dispatcher-servlet为你的springmvc的配置文件名），它是Root WebApplicationContext的子容器。

解决办法就是判断当前WebApplicationContext是否有父容器。

{% highlight java %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

/**
 * 在spring启动后执行一些操作
 */
@Component
public class StartUpListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {

        if(event.getApplicationContext().getParent() == null) {//父容器为null就表示是root applicationContext
            // TODO
        }
    }
}
{% endhighlight %}
