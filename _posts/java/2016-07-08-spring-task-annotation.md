---
layout: post
title:  "Spring Task 定时任务的注解实现"
date:   2016-06-22 13:20:53 +0800
categories: java
tags: [Spring Task]
---
> 1、引入相关jar包。
> 2、在Spring配置文件中加入“<task:annotation-driven/>”，并配置扫描路径。

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">

    ......

    <task:annotation-driven/>

    <!-- 注入@Service, @Repository -->
    <context:component-scan base-package="com.ces">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    ......

</beans>
{% endhighlight %}

> 3、实现类加上“@Component”注解，方法加上“@Scheduled”注解并配置cron表达式。

{% highlight xml %}
@Component
public class PresaleProductTask {
    /**
     * 每天0点执行
     */
    @Scheduled(cron="0 0 0 * * ?")
    public void excute() {
      //TODO 具体任务内容
    }
}
{% endhighlight %}

> 4、cron表达式语法为：

```
Seconds Minutes Hours DayofMonth Month DayofWeek Year
或
Seconds Minutes Hours DayofMonth Month DayofWeek
```
