---
title: tomcat修改session默认的cookie名字
categories: tomcat
tags: tomcat
date: 2019-07-17 20:07:08
---

Tomcat7修改
----------

在tomcat的conf目录下, 修改`server.xml`文件, 在<Host>节点中加入<Context>配置:

```xml
<Context path="/" sessionCookiePath="/" sessionCookieName="MY-SESSION"/>
```