---
title: 初识JMX
categories: JMX
tags: JMX
date: 2019-06-15 20:47:00
---

JXM简介
----------

试想，一个正在运行中的程序，我们如果想改变程序中的一些属性，可以通过什么方法呢？可能有这么几个方法：

- 对于服务器式的程序，可以制作管理页面，通过HTTP post与servlet来更改服务器端程序的属性。
- 对于服务器式的程序，还可以通过SOAP方式。但这需要程序开启了SOAP端的服务。
- 可以使用RMI远程调用。但这需要设计开启RMI服务。
- 如果是SWT或Swing的程序，则可以通过设计UI管理界面，使用户可以和程序内部交互。
- 还有一种方式，是将可改变的属性放入配置文件XML，properties或数据库，程序轮询配置文件，以求获取最新的配置。

上面几个方法都是常见，但却无法通用的。所谓通用，是指解决方案符合一个标准，使得任何符合此标准的工具都能解析针对此标准的方案实现。这样A公司设计的方案，B公司可以根据标准来解析。JMX就是Java管理标准。

对于一些参数的修改，网上有一段描述还是比较形象的：
1. 程序初哥一般是写死在程序中，到要改变的时候就去修改代码，然后重新编译发布。
2. 程序熟手则配置在文件中（JAVA一般都是properties文件），到要改变的时候只要修改配置文件，但还是必须重启系统，以便读取配置文件里最新的值。
3. 程序好手则会写一段代码，把配置值缓存起来，系统在获取的时候，先看看配置文件有没有改动，如有改动则重新从配置里读取，否则从缓存里读取。
4. 程序高手则懂得物为我所用，用JMX把需要配置的属性集中在一个类中，然后写一个MBean，再进行相关配置。另外JMX还提供了一个工具页，以方便我们对参数值进行修改。

JMX(Java Management Extensions)是一个为应用程序植入管理功能的框架。JMX是一套标准的代理和服务，实际上，用户可以在任何Java应用程序中使用这些代理和服务实现管理。JMX让程序有被管理的功能，例如你开发一个WEB网站，它是在24小时不间断运行，那么你肯定会对网站进行监控，如每天的UV、PV是多少；又或者在业务高峰的期间，你想对接口进行限流，就必须去修改接口并发的配置值。

应用场景：中间件软件WebLogic的管理页面就是基于JMX开发的，而JBoss则整个系统都基于JMX构架。

JMX的构成
--------------

JMX由三部分组成：

- 基础层：程序端的Instrumentation, 我把它翻译成可操作的仪器。这部分就是指的MBean. MBean类似于JavaBean。最常用的MBean则是Standard MBean和MXBean.

- 适配层：程序端的JMX agent. 这部分指的是MBean Server. MBean Server则是启动与JVM内的基于各种协议的适配器。用于接收客户端的调遣，然后调用相应的MBeans.

- 接入层：客户端的Remote Management. 这部分则是面向用户的程序。此程序则是MBeans在用户前投影，用户操作这些投影，可以反映到程序端的MBean中去。这内部的原理则是client通过某种协议调用agent操控MBeans. 
JMX agent与Remote Management之间是通过协议链接的，这协议可能包含：
    - HTTP
    - SNMP
    - RMI
    - IIOP

JMX agent中有针对上面协议的各种适配器。可以解析通过相应协议传输过来的数据。Remote Management client则可以用现成的工具，如JConsole, 也可以自己书写java code。

实现一个JMX程序
--------------------

 1、 首先定义一个MBean接口，接口的命名规范为以具体的实现类为前缀（这个规范很重要）

```java
public interface HelloMBean {
     public String getName();
     public void setName(String name);
     public String getAge();
     public void setAge(String age);
     public void helloWorld();
     public void helloWorld(String str);
     public void getTelephone();
}
```

2、定义一个实现类，实现上面的接口：

```java
/*
 * 该类名称必须与实现的接口的前缀保持一致（即MBean前面的名称
 */
public class Hello implements HelloMBean {
    private String name;
    private String age;

    public void getTelephone() {
        System.out.println("get Telephone");
    }

    public void helloWorld() {
        System.out.println("hello world");
    }

    public void helloWorld(String str) {
        System.out.println("helloWorld:" + str);
    }

    public String getName() {
        System.out.println("get name 123");
        return name;
    }

    public void setName(String name) {
        System.out.println("set name 123");
        this.name = name;
    }

    public String getAge() {
        System.out.println("get age 123");
        return age;
    }

    public void setAge(String age) {
        System.out.println("set age 123");
        this.age = age;
    }      
}
```

3、定义agent层：

```java
import java.lang.management.ManagementFactory;
import javax.management.JMException;
import javax.management.MBeanServer;
import javax.management.ObjectName;

public class HelloAgent {
    public static void main(String[] args) throws JMException, Exception {
          // 通过工厂类获取MBeanServer，用来做MBean的容器 
         MBeanServer server = ManagementFactory.getPlatformMBeanServer();
          // ObjectName中的取名是有一定规范的，格式为：“域名：name=MBean名称”，其中域名和MBean的名称可以任意取。
         ObjectName helloName = new ObjectName("jmxBean:name=hello");
         //将Hello这个类注入到MBeanServer中，注入需要创建一个ObjectName类
         server.registerMBean(new Hello(), helloName);
         Thread.sleep(60*60*1000);
    }
}
```

这样，一个简单的JMX的DEMO已经写完了，现在我们通过JDK提供的Jconsole来进行操作。

4、在JDK安装路径 ·JAVA_HOME\bin· 下找到 jconsole.exe 这个小工具，双击打开。

在本地进程中找到 `HelloAgent` 并双击打开

在当前界面上，我们可以给程序中HelloMBean的属性赋值，也可以调用其中的方法

这样就做到动态修改运行中程序的状态进而管理程序。
