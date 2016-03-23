---
layout: post
title:  "Java BigDecimal 处理高精度计算"
date:   2016-03-23 15:17:19 +0800
categories: java
---
Java的Double和Float类型在做运算的时候会出现丢失精度问题，比如：

{% highlight java %}
System.out.println(4.8D+3); //7.8
System.out.println(4.8D-3); //1.7999999999999998
System.out.println(4.8D*3); //14.399999999999999
System.out.println(4.8D/3); //1.5999999999999999
System.out.println(4.8F>4.8D); //true
{% endhighlight %}

此时我们可以使用“java.math.BigDecimal”来进行运算：

{% highlight java %}
System.out.println(new BigDecimal(String.valueOf(4.8D)).add(new BigDecimal(3))); //7.8
System.out.println(new BigDecimal(String.valueOf(4.8D)).subtract(new BigDecimal(3))); //1.8
System.out.println(new BigDecimal(String.valueOf(4.8D)).multiply(new BigDecimal(3))); //14.4
System.out.println(new BigDecimal(String.valueOf(4.8D)).divide(new BigDecimal(3))); //1.6
System.out.println(new BigDecimal(String.valueOf(4.8F)).compareTo(new BigDecimal(String.valueOf(4.8D)))); //0
{% endhighlight %}

“compareTo(BigDecimal val)” 方法的返回值：1(大于)，-1(小于), 0(等于)。

需要注意的是，在实例化“BigDecimal”的时候，如果是浮点型，需要传入字符型的参数，即调用“public BigDecimal(String val)”方法来构造。
