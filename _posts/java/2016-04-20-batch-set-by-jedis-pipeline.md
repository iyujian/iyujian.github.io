---
layout: post
title:  "Jedis如何快速批量插入大量数据"
date:   2016-04-20 18:10:32 +0800
categories: java
tags: [redis]
---
Jedis可以通过“redis.clients.jedis.Pipeline”来高效的批量插入大量数据，直接上代码：

{% highlight java %}
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

public class TestRedis {

    private static final String REDIS_IP = "127.0.0.1";

    private static Jedis jedis = new Jedis(REDIS_IP);

    private static Pipeline pipeline = jedis.pipelined();

    private static int count = 10000;

    /**
     * 循环插入数据
     */
    public static void singleHset() {
        for(int i=0; i<count; i++) {
            jedis.hset("sequence_single", ""+i, ""+i);
        }
    }

    /**
     * 批量插入数据
     */
    public static void batchHset() {
        for(int i=0; i<count; i++) {
            pipeline.hset("sequence_batch", ""+i, ""+i);
        }
        pipeline.sync();
    }

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();
        singleHset();
        System.out.println("循环插入耗时：" + (System.currentTimeMillis() - startTime));
        startTime = System.currentTimeMillis();
        batchHset();
        System.out.println("批量插入耗时：" + (System.currentTimeMillis() - startTime));
    }

}
{% endhighlight %}

输出为：

{% highlight text %}
循环插入耗时：22101
批量插入耗时：156
{% endhighlight %}

从数据可以看出，使用 Pipeline 来做数据批量插入效率还是很高的。
