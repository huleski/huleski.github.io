---
title: Tomcat配置
categories: tomcat
tags: tomcat
date: 2019-07-10 11:43:47
---

一台机器启动多个tomcat
---------

从[Apache官网](https://tomcat.apache.org/download-80.cgi)下载好Tomcat, 解压两份到不同的文件夹下即可

第一个tomcat不做任何修改，使用默认端口和配置

第二个tomcat需要编辑tomcat/conf目录下server.xml文件, 修改3个端口, 使其与前面的tomcat端口不同:

```xml
<!-- 修改关闭端口为8015, 第22行左右 -->
<Server port="8015" shutdown="SHUTDOWN">

......

<!-- 修改服务端口为8081, 第70行左右 -->
<Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

......

<!-- 修改连接端口为8019, 第91行左右 -->
<Connector port="8019" protocol="AJP/1.3" redirectPort="8443" />
```

配置完成, 可以去启动两个tomcat了

看网上有些说还需要改redirectPort, 设置啥CATALINA_HOME等等, 都是多余的步骤, 完全不用

指定JDK版本启动tomcat
-----------------

有些复杂环境中有多个JDK, 默认会使用系统环境中的`JAVA_HOME`来启动tomcat, 如需要指定不同JDK版本启动tomcat, 这就需要修改tomcat的启动配置了

>在Windows中启动Tomcat时，双击startup.bat然后会调用catalina.bat文件，而catalina.bat会调用setclasspath.bat文件来获取JAVA_HOME和JRE_HOME这两个环境变量的值，因此只要在tomcat启动时指向特定的JDK即可

- 在Windows中, 需要在`setclasspath.bat`文件的开头处加入以下内容

```bat
set JAVA_HOME=C://Program Files/Java/jdk1.7.0_79
set JRE_HOME=C://Program Files/Java/jdk1.7.0_79/jre
```

- 在Linux中, 则需要在`setclasspath.sh`文件的开头处加入以下内容

```bash
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
export JRE_HOME=/usr/local/java/jdk1.7.0_79/jre
```

修改session默认的cookie名字
----------

在tomcat的conf目录下, 修改`server.xml`文件, 在<Host>节点中加入<Context>配置:

```xml
<Context path="/" sessionCookiePath="/" sessionCookieName="MY-SESSION"/>
```

console控制台输出有乱码
--------------------

在tomcat安装路径`conf/logging.properties`文件中注释掉其中一行

```property
## 将下面这行注释掉
# java.util.logging.ConsoleHandler.encoding = UTF-8
```
