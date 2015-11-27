---
layout: post
title:  "DateConverter does not support default String to 'Date' conversion"
date:   2015-11-27 11:37:16 +0800
categories: java
tags: spring
---
先贴上错误信息：

```
严重: Servlet.service() for servlet [dispatcher] in context with path [] threw exception [Request processing failed; nested exception is org.apache.commons.beanutils.ConversionException: DateConverter does not support default String to 'Date' conversion.] with root cause
org.apache.commons.beanutils.ConversionException: DateConverter does not support default String to 'Date' conversion.
	at org.apache.commons.beanutils.converters.DateTimeConverter.toDate(DateTimeConverter.java:474)
	at org.apache.commons.beanutils.converters.DateTimeConverter.convertToType(DateTimeConverter.java:347)
	at org.apache.commons.beanutils.converters.AbstractConverter.convert(AbstractConverter.java:169)
	at org.apache.commons.beanutils.converters.ConverterFacade.convert(ConverterFacade.java:61)
	at org.apache.commons.beanutils.ConvertUtilsBean.convert(ConvertUtilsBean.java:566)
	at org.apache.commons.beanutils.ConvertUtils.convert(ConvertUtils.java:282)
	...
```

出现这个异常信息，是因为 org.apache.commons.beanutils.ConvertUtils.convert(Object value, Class<?> targetType) 方法不支持 String 转 java.util.Date。 追踪源码我们可以发现它支持的转化类型只有：

<ul>
 <li>java.lang.BigDecimal (no default value)</li>
 <li>java.lang.BigInteger (no default value)</li>
 <li>boolean and java.lang.Boolean (default to false)</li>
 <li>byte and java.lang.Byte (default to zero)</li>
 <li>char and java.lang.Character (default to a space)</li>
 <li>java.lang.Class (no default value)</li>
 <li>double and java.lang.Double (default to zero)</li>
 <li>float and java.lang.Float (default to zero)</li>
 <li>int and java.lang.Integer (default to zero)</li>
 <li>long and java.lang.Long (default to zero)</li>
 <li>short and java.lang.Short (default to zero)</li>
 <li>java.lang.String (default to null)</li>
 <li>java.io.File (no default value)</li>
 <li>java.net.URL (no default value)</li>
 <li>java.sql.Date (no default value)</li>
 <li>java.sql.Time (no default value)</li>
 <li>java.sql.Timestamp (no default value)</li>
 </ul>

解决方案：

org.apache.commons.beanutils.ConvertUtils 提供了一个 register(Converter converter, Class<?> clazz) 方法，使用这个方法可以注册我们自己的类型转换器。

{% highlight java %}
ConvertUtils.register(new DateLocaleConverter(), Date.class);
{% endhighlight %}

经测试，DateLocaleConverter() 只可以实现对日期（不包含时分秒）的转换。 如果要支持带时分秒的，我们就要自己去实现转换器了，并在 Spring 启动后执行注册的操作。

{% highlight java linenos %}
import org.apache.commons.beanutils.ConvertUtils;
import org.apache.commons.beanutils.Converter;
import org.apache.commons.lang3.time.DateUtils;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

@Component
public class ConverterListener implements ApplicationListener<ContextRefreshedEvent> {

    private static final String DATETIME_PATTERN = "yyyy-MM-dd HH:mm:ss";

    private static final String DATE_PATTERN = "yyyy-MM-dd";

    private static final String MONTH_PATTERN = "yyyy-MM";

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        if (event.getApplicationContext().getParent() == null) {//父容器为null就表示是root applicationContext

            SimpleDateFormat sd = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

            ConvertUtils.register(new Converter() {
                @Override
                public <T> T convert(Class<T> type, Object value) {

                    try {
                        Date date = DateUtils.parseDate((String) value, new String[]{DATETIME_PATTERN, DATE_PATTERN, MONTH_PATTERN});
                        return (T) date;
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }

                    return null;
                }
            }, Date.class);
        }
    }
}
{% endhighlight %}