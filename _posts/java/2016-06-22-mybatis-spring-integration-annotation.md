---
layout: post
title:  "Mybatis和Spring的整合（注解）"
date:   2016-06-22 13:20:53 +0800
categories: java
tags: [mybatis]
---

> 1、添加“mybatis”和“mybatis-spring”的依赖。

{% highlight xml %}
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.0</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
{% endhighlight %}

> 2、添加Mybatis的配置文件：mybatis-config.xml。

更多配置请参考Mybatis官方中文文档：http://www.mybatis.org/mybatis-3/zh/configuration.html#settings。

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--该配置影响的所有映射器中配置的缓存的全局开关。默认值：true。-->
        <setting name="cacheEnabled" value="true"/>
        <!-- 延迟加载的全局开关。默认值：false。 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 是否允许单一语句返回多结果集（需要兼容驱动）。默认值：true。-->
        <setting name="multipleResultSetsEnabled" value="true"/>
        <!-- 是否使用列标签代替列名。默认值：true。-->
        <setting name="useColumnLabel" value="true"/>
        <!-- 是否允许 JDBC 支持自动生成主键，需要驱动兼容。默认值：false。 -->
        <setting name="useGeneratedKeys" value="false"/>
        <!-- 指定 MyBatis 应如何自动映射列到字段或属性。默认值：PARTIAL。-->
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <!-- Specify the behavior when detects an unknown column (or unknown property type) of automatic mapping target. 默认值：NONE。-->
        <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
        <!-- 配置默认的执行器。默认值：SIMPLE。-->
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <!-- 设置超时时间，它决定驱动等待数据库响应的秒数。-->
        <setting name="defaultStatementTimeout" value="25"/>
        <!-- Sets the driver a hint as to control fetching size for return results. -->
        <setting name="defaultFetchSize" value="100"/>
        <!-- 允许在嵌套语句中使用分页。默认值：false。-->
        <setting name="safeRowBoundsEnabled" value="false"/>
        <!-- 是否开启自动驼峰命名规则（camel case）映射。默认值：false。-->
        <setting name="mapUnderscoreToCamelCase" value="false"/>
        <!-- MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值：SESSION。-->
        <setting name="localCacheScope" value="SESSION"/>
        <!-- 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。默认值：OTHER。 -->
        <setting name="jdbcTypeForNull" value="OTHER"/>
        <!-- 指定哪个对象的方法触发一次延迟加载。默认值：equals,clone,hashCode,toString。-->
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>
</configuration>
{% endhighlight %}

> 3、数据源的配置略过...，下面是SessionFactory和Spring事务的相关配置

{% highlight xml %}
<!-- Mybatis SqlSessionFactory-->
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
</bean>
<!-- Mybatis 映射器扫描配置，用于自动查找类路径下的映射器并自动将它们创建成MapperFactoryBean。-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.devaeye.cms.core.dao"/>
</bean>

<!-- 事务管理 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
{% endhighlight %}

> 4、以上为所有xml部分的配置，下面为Java部分。

>> 4.1、UserEntity.java

{% highlight java %}
public class UserEntity {

    private Long id;

    private String username;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
{% endhighlight %}

>> 4.2、UserService.java

{% highlight java %}
@Service
@Transactional(readOnly = true)
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Transactional(readOnly = false)
    public void save(UserEntity entity) {
        entity.setUsername("username");
        userMapper.insert(entity);
    }
}
{% endhighlight %}

>> 4.3、UserMapper.java

{% highlight java %}
import com.devaeye.cms.core.entity.UserEntity;
import org.apache.ibatis.annotations.Insert;

public interface UserMapper {

    @Insert("INSERT INTO cms_user (username, PASSWORD, salt, email) VALUES (#{username}, #{password}, #{salt}, #{email})")
    void insert(UserEntity userEntity);
}
{% endhighlight %}

> 5、The end。
