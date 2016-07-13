---
layout: post
title:  "如何创建Core - Apache Solr"
date:   2016-07-13 10:58:49 +0800
categories: java
tags: [Apache Solr]
---

Apache Solr 版本：6.1.0。

> 1、按上一篇：["Apache Solr 6.1.0 在 Tomcat 中部署运行"](deploy-apache-solr-in-tomcat.html "Apache Solr 6.1.0 在 Tomcat 中部署运行")，先让Solr运行起来。

> 2、在“solr_home”下新建文件夹“new_core”，将“solr-6.1.0/example/example-DIH/solr/solr”下的所有文件复制到该文件夹中，重启Tomcat，稍等一会儿，Solr会自动扫描到该Core，名称为“new_core”。
