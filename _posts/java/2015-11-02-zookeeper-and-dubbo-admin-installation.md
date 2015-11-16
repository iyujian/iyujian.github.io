---
layout: post
title:  "基于ZooKeeper的Dubbo注册中心安装(CentOS)"
date:   2015-11-02 09:27:08 +0800
categories: java
tags: zookeeper dubbo
---

DUBBO是一个分布式服务框架，用于提供高性能和透明化的RPC远程服务调用方案，是阿里巴巴SOA服务化治理方案的核心框架。Dubbo架构的节点包含Provider、Consumer、Registry、Monitor、Container，其中Registry（注册中心）可以使用Zookeeper注册中心、Redis注册中心等来进行服务注册。下面是Zookeeper及Dubbo-admin的安装过程：

<!-- more -->

1、安装jdk并配置java环境变量

2、安装Zookeeper

(1) 下载Zookeeper

{% highlight bash %}
wget http://www.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
{% endhighlight %}

(2) 解压

{% highlight bash %}
tar zxvf zookeeper-3.4.6.tar.gz
{% endhighlight %}

(3) 复制 conf/zoo_sample.cfg 为 conf/zoo.cfg

{% highlight bash linenos %}
cd zookeeper-3.4.6/conf
cp zoo_sample.cfg zoo.cfg
{% endhighlight %}

(4) 对zoo.cfg文件做相应修改

tickTime：每次心跳的毫秒数。Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。默认情况下最小的会话超时时间为两倍的tickTime。

dataDir：快照的目录。Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。

clientPort：客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

initLimit：LF初始通信时限。集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。

syncLimit：LF同步通信时限。集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。

maxClientCnxns：限制连接到 ZooKeeper 的客户端的数量。

更多配置项查看：http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance

(5) 启动

{% highlight bash %}
../bin/zkServer.sh start
{% endhighlight %}

到此，Zookeeper已经配置完成。

安装dubbo（官网：http://dubbo.io）：

3、下载并安装tomcat

{% highlight bash linenos %}
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.65/bin/apache-tomcat-7.0.65.tar.gz
tar zxvf apache-tomcat-6.0.35.tar.gz
cd apache-tomcat-6.0.35
rm -rf webapps/ROOT
{% endhighlight %}

4、安装dubbo-admin

(1) 下载并解压dubbo-admin

{% highlight bash linenos %}
wget http://code.alibabatech.com/mvn/releases/com/alibaba/dubbo-admin/2.4.9/dubbo-admin-2.4.9.war
unzip dubbo-admin-2.4.1.war -d webapps/ROOT
{% endhighlight %}

ps：现在http://code.alibabatech.com已打不开，可以去https://github.com/alibaba/dubbo下载源码，并自己打成war包。

(2) 配置: (或将dubbo.properties放在当前用户目录下)

{% highlight bash %}
vi webapps/ROOT/WEB-INF/dubbo.properties
{% endhighlight %}

修改相应配置如下：

{% highlight text linenos %}
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
{% endhighlight %}

(3) 启动:

{% highlight bash %}
./bin/startup.sh
{% endhighlight %}

(4) 停止：

{% highlight bash %}
./bin/shutdown.sh
{% endhighlight %}

5、修改iptables，解除端口相应端口限制。


