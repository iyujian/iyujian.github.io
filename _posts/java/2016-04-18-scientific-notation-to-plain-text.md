---
layout: post
title:  "如何将科学记数法形式转换成普通数字形式显示"
date:   2016-04-18 12:56:50 +0800
categories: java
---
Java中，数字小数点位数太多，或者数字太大，会显示成科学记数法的形式，比如：

{% highlight java %}
System.out.println("0.00000000001d >> " + 0.00000000001d);
System.out.println("0.00000000001f >> " + 0.00000000001f);
System.out.println("0.00000000001 >> " + 0.00000000001);
System.out.println("1000000000d >> " + 1000000000d);
System.out.println("1000000000f >> " + 1000000000f);
System.out.println("1000000000 >> " + 1000000000);
{% endhighlight %}

运行结果为：

```
0.00000000001d >> 1.0E-11
0.00000000001f >> 1.0E-11
0.00000000001 >> 1.0E-11
1000000000d >> 1.0E9
1000000000f >> 1.0E9
1000000000 >> 1000000000
```

这种情况可以使用“java.math.BigDecimal”来进行转换：

{% highlight java %}
public static String format(Number value) {
    return new BigDecimal(String.valueOf(value)).stripTrailingZeros().toPlainString();
}
{% endhighlight %}

然后调用format()方法来进行转换：

{% highlight java %}
System.out.println("0.00000000001d >> " + format(0.00000000001d));
System.out.println("0.00000000001f >> " + format(0.00000000001f));
System.out.println("0.00000000001 >> " + format(0.00000000001));
System.out.println("1000000000d >> " + format(1000000000d));
System.out.println("1000000000f >> " + format(1000000000f));
System.out.println("1000000000 >> " + format(1000000000));
{% endhighlight %}

结果为：

```
0.00000000001d >> 0.00000000001
0.00000000001f >> 0.00000000001
0.00000000001 >> 0.00000000001
1000000000d >> 1000000000
1000000000f >> 1000000000
1000000000 >> 1000000000
```

stripTrailingZeros()方法返回移除多余的尾部零后的值。

toPlainString()方法返回非指数形式的值的字符串。