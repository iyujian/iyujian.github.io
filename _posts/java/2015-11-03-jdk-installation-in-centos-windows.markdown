---
layout: post
title:  "JDK的安装和Java环境变量的配置(CentOS和windows)"
date:   2015-11-03 14:09:27 +0800
categories: java
---
1、从oracle下载JDK并安装，windows下直接点击安装即可，CentOS下用yum或rpm安装，具体过程略过。

2、Windows下Java环境变量的配置

1) 右击“计算机” -> “属性” -> “高级系统设置” -> “高级” -> “环境变量”；

2) 新建系统变量“JAVA_HOME”，值为：jdk的安装路径，比如“D:\Java\jdk1.6.0_26”；

3) 新建系统变量“CLASS_PATH”，值为：.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar（tools.jar和dt.jar的路径）；

4) 修改系统变量“Path”，值为：;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin（bin和jre\bin的路径）；

5) “javac”，显示javac命令用法等信息，表示java环境变量配置成功。

3、CentOS下Java环境变量的配置

{% highlight shell linenos %}

vi /etc/profile

{% endhighlight %}

加入如下配置：

{% highlight plaintext linenos %}

export JAVA_HOME=/usr/java/jdk1.7.0_79
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

{% endhighlight %}




