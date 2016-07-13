---
layout: post
title:  "Apache Solr 6.1.0 在 Tomcat 中部署运行 - Apache Solr"
date:   2016-07-11 15:34:13 +0800
categories: java
tags: [Apache Solr]
---
> 1、Apache Solr官方下载地址：http://www.apache.org/dyn/closer.lua/lucene/solr/6.1.0。当前最新版本为：6.1.0。

> 2、Apache Solr 6.1.0环境要求：JDK8及以上。

> 3、Apache Solr 6.1.0已内置jetty，可以通过运行命令：bin/solr start -e cloud -noprompt，直接启动Solr，默认访问端口：8983，具体过程可以参考Solr官方文档：http://lucene.apache.org/solr/quickstart.html。此文主要讲解如何将Solr部署在Tomcat中运行。

> 4、解压下载的Solr压缩包，将“solr-6.1.0/server/solr-webapp/webapp”复制到Tomcat下的webapps目录下，并重命名为“solr”。

> 5、将“solr-6.1.0/server/lib/ext”下的所有文件复制到“solr/WEB-INF/lib”下，再将“solr-6.1.0/dist”下的solr-dataimporthandler-6.1.0.jar 和 solr-dataimporthandler-extras-6.1.0.jar也复制过去。

> 6、将“solr-6.1.0/server/solr”文件夹复制到“solr”下（可以是任意目录，只需要和第7条的”env-entry-value”保持一致即可），并重命名为“solr_home”。

> 7、修改web.xml文件中“env-entry”，并取消注释。此处配置”env-entry-value”即为第6条的文件夹地址。

{% highlight xml %}
<env-entry>
   <env-entry-name>solr/home</env-entry-name>
   <env-entry-value>.../tomcat-solr/webapps/solr/solr_home</env-entry-value>
   <env-entry-type>java.lang.String</env-entry-type>
</env-entry>
{% endhighlight %}

> 8、在“solr/WEB-INF/”下新建“classes”文件夹，并将“solr-6.1.0/server/resources/log4j.properties”文件复制到该文件夹中。

> 9、启动Tomcat，并访问：http://localhost:8080/solr/index.html。（注意要带上index.html，不然就404。）
