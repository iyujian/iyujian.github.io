---
layout: post
title:  "Jedis 如何缓存对象及对象的List"
date:   2015-10-30 19:44:46 +0800
categories: java
tags: redis
---
Jedis 目前并没有提供对Java对象的缓存和读取方法，但是它为我们提供了这个方法：

{% highlight java %}

jedis.set(byte[] key, byte[] value);

{% endhighlight %}

我们可以使用这个方法来间接的实现对对象的缓存。原理就是在缓存对象之前，先将该对象序列化为 byte 数组，而取出缓存后再反序列化为对象。

<!-- more -->

1、序列化工具抽象类

{% highlight java %}

import java.io.*;

/**
 * 序列化工具抽象类
 */
public abstract class SerializationUtil {

    /**
     * 序列化方法
     * @param object
     * @return
     */
    public abstract byte[] serialize(Object object);

    /**
     * 反序列化方法
     * @param bytes
     * @return
     */
    public abstract Object deserialize(byte[] bytes);

    /**
     * 关闭流
     * @param closeable
     */
    public void close(Closeable closeable) {
        if (closeable != null) {
            try {
                closeable.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}

{% endhighlight %}

2、对象序列化工具类

{% highlight java %}

import java.io.*;

/**
 * 对象序列化工具类
 */
public class ObjectSerializationUtil<T extends Serializable> extends SerializationUtil {

    @Override
    public byte[] serialize(Object object) {
        ByteArrayOutputStream baos = null;
        ObjectOutputStream oos = null;
        baos = new ByteArrayOutputStream();
        try {
            oos = new ObjectOutputStream(baos);
            T t = (T) object;
            oos.writeObject(t);
            byte[] bytes = baos.toByteArray();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            close(oos);
            close(baos);
        }
        return null;
    }

    @Override
    public T deserialize(byte[] bytes) {
        ByteArrayInputStream bais = null;
        ObjectInputStream ois = null;
        bais = new ByteArrayInputStream(bytes);
        try {
            ois = new ObjectInputStream(bais);
            return (T) ois.readObject();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            close(ois);
            close(bais);
        }
        return null;
    }
}

{% endhighlight %}

3、对象List序列化工具类

{% highlight java %}

import java.io.*;
import java.util.ArrayList;
import java.util.List;

/**
 * 对象List序列化工具类
 */
public class ListSerializationUtil<T extends Serializable> extends SerializationUtil {


    @Override
    public byte[] serialize(Object object) {
        List<T> list = (List<T>) object;

        ByteArrayOutputStream baos = null;
        ObjectOutputStream oos = null;
        baos = new ByteArrayOutputStream();
        try {
            oos = new ObjectOutputStream(baos);
            for(T t : list) {
                oos.writeObject(t);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            close(oos);
            close(baos);
        }

        return null;
    }

    @Override
    public List<T> deserialize(byte[] bytes) {

        ByteArrayInputStream bais = null;
        ObjectInputStream ois = null;
        bais = new ByteArrayInputStream(bytes);
        try {
            ois = new ObjectInputStream(bais);
            List<T> list = new ArrayList<T>();
            while(true) {
                T t  = (T) ois.readObject();
                if(null == t) {
                    break;
                }
                list.add(t);
            }
            return list;
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            close(ois);
            close(bais);
        }

        return null;
    }
}

{% endhighlight %}

4、Jedis工具类

{% highlight java %}

import java.io.Serializable;
import java.util.List;

import redis.clients.jedis.ShardedJedis;
import redis.clients.jedis.ShardedJedisPool;

/**
 * Jedis工具类
 */
public class JedisUtil<T extends Serializable> {
	
    private static ShardedJedisPool pool;
    
    private ObjectSerializationUtil<T> osu = new ObjectSerializationUtil<T>();
    
    private ListSerializationUtil<T> lsu = new ListSerializationUtil<T>();

    /**
     * get ShardedJedis
     *
     * @return ShardedJedis
     */
    public static ShardedJedis getResource() {
        return pool.getResource();
    }
    
    /**
     * 缓存对象到Redis
     * @param key
     * @param value
     */
    public void setObject(String key, T value) {
    	byte[] bytes = osu.serialize(value);
    	ShardedJedis jedis = getResource();
    	try {
            jedis.set(key.getBytes(), bytes);
        } finally {
            pool.returnResourceObject(jedis);
        }
    }
    
    /**
     * 从Redis中获取缓存对象
     * @param key
     * @return
     */
    public T getObject(String key) {
    	ShardedJedis jedis = getResource();
    	try {
            return osu.deserialize(jedis.get(key.getBytes()));
        } finally {
            pool.returnResourceObject(jedis);
        }
    }
    
    /**
     * 缓存对象List到Redis
     * @param key
     * @param value
     */
    public void setList(String key, T value) {
    	byte[] bytes = lsu.serialize(value);
    	ShardedJedis jedis = getResource();
    	try {
            jedis.set(key.getBytes(), bytes);
        } finally {
            pool.returnResourceObject(jedis);
        }
    }
    
    /**
     * 从Redis中获取对象List
     * @param key
     * @return
     */
    public List<T> getList(String key) {
    	ShardedJedis jedis = getResource();
    	try {
            return lsu.deserialize(jedis.get(key.getBytes()));
        } finally {
            pool.returnResourceObject(jedis);
        }
    }
    
    public void setPool(ShardedJedisPool pool) {
        JedisUtil.pool = pool;
    }

}

{% endhighlight %}

5、配置文件

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.jedisPoolConfig.maxTotal}"/>
        <property name="maxIdle" value="${redis.jedisPoolConfig.maxIdle}"/>
        <property name="maxWaitMillis" value="${redis.jedisPoolConfig.maxWaitMillis}"/>
        <property name="testOnBorrow" value="${redis.jedisPoolConfig.testOnBorrow}"/>
    </bean>

    <!-- Redis 服务器配置  -->
    <bean id="jedisShardInfo" class="redis.clients.jedis.JedisShardInfo">
        <constructor-arg index="0" value="${jedis.host}"/>
        <constructor-arg index="1" type="int" value="${jedis.port}"/>
    </bean>

    <!-- Redis 资源池  -->
    <bean id="shardedJedisPool" class="redis.clients.jedis.ShardedJedisPool">
        <constructor-arg index="0" ref="jedisPoolConfig"/>
        <constructor-arg index="1">
            <list>
                <ref bean="jedisShardInfo"/>
            </list>
        </constructor-arg>
    </bean>

    <bean class="com.***.JedisUtil">
        <property name="pool" ref="shardedJedisPool"/>
    </bean>
</beans>

{% endhighlight %}


