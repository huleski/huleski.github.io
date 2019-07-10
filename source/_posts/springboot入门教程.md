---
title: springboot入门教程
date: 2019-01-31 16:45:42
toc: true #是否显示文章目录
categories: springboot #分类
tags:   #标签
	- springboot
---

构建微服务：Spring boot 入门篇
===========

什么是spring boot
-------------

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是spring boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了所有的框架（有点夸张）。

使用spring boot有什么好处
--------

其实就是简单、快速、方便！平时如果我们需要搭建一个spring web项目的时候需要怎么做呢？
1. 配置web.xml，加载spring和spring mvc
2. 配置数据库连接、配置spring事务
3. 配置加载配置文件的读取，开启注解
4. 配置日志文件

快速入门
=========

说了那么多，手痒痒的很，马上来一发试试!

maven构建项目
--------

1. 访问http://start.spring.io/
2. 选择构建工具Maven Project、Spring Boot版本1.3.6以及一些工程基本信息，点击“Switch to the full version.”java版本选择1.7。
3. 点击Generate Project下载项目压缩包
4. 解压后，使用IDEA Import

项目结构介绍
--------

Spring Boot的基础结构共三个文件：

```
l src/main/java  程序开发以及主程序入口
l src/main/resources 配置文件
l src/test/java  测试程序
```

另外，spingboot建议的目录结果如下：
root package结构：com.example.myproject

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |
```

1. Application.java 建议放到跟目录下面,主要用于做一些框架配置
2. domain目录主要用于实体（Entity）与数据访问层（Repository）
3. service 层主要是业务类代码
4. controller 负责页面访问控制

采用默认配置可以省去很多配置，当然也可以根据自己的喜欢来进行更改。最后，启动Application main方法，至此一个java项目搭建好了！

引入web模块

1. pom.xml中添加支持web的模块：

```
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
 </dependency>
```

pom.xml文件中默认有两个模块：

spring-boot-starter：核心模块，包括自动配置支持、日志和YAML；
spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito。

2. 编写controller内容
```
@RestController
public class HelloWorldController {
    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
}
```

@RestController的意思就是controller里面的方法都以json格式输出，不用再写什么jackjson配置的了！

3. 启动主程序，打开浏览器访问http://localhost:8080/hello，就可以看到效果了，有木有很简单！

如何做单元测试
------

打开的src/test/下的测试入口，编写简单的http请求来测试；使用mockmvc进行，利用MockMvcResultHandlers.print()打印出执行结果。
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MockServletContext.class)
@WebAppConfiguration
public class HelloWorldControlerTests {
    private MockMvc mvc;
    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
    }
    @Test
    public void getHello() throws Exception {
    mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
    }
}
```

开发环境的调试
----------

热启动在正常开发项目中已经很常见了吧，虽然平时开发web项目过程中，改动项目启重启总是报错；修改之后可以实时生效，需要添加以下的配置：
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
   </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
   </plugins>
</build>
```