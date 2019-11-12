---
title: java 8 本新特性
categories: java
tags: java
date: 2019-11-12 20:17:25
---

[官网介绍](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)

参考: [【译】Java 8的新特性—终极版](https://www.jianshu.com/p/5b800057f2d8)

- Lambda 表达式 和 函数式接口 − Lambda 允许把函数作为一个方法的参数（函数作为参数传递到方法中）。

- 方法引用 − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

- 默认方法 − 默认方法就是一个在接口里面有了一个实现的方法。

- 应用范围扩大的注解, 可以重复注解, 更好的类型推断

- 新工具 − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。

- Stream API −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。

- Date Time API − 加强对日期与时间的处理。

- Optional 类 − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。

- Nashorn, JavaScript 引擎 − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。

- JVM的新特性 - 使用Metaspace（JEP 122）代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize。
