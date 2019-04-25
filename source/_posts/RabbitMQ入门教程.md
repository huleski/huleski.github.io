---
title: RabbitMQ入门教程
categories: RabbitMQ
tags: RabbitMQ
date: 2019-04-19 10:25:59
---

RabbitMQ 简介
-----------

RabbitMQ是一个在AMQP（Advanced Message Queuing Protocol ）基础上实现的，可复用的企业消息系统。它可以用于大型软件系统各个模块之间的高效通信，支持高并发，支持可扩展。

AMQP
----

即Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准,为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

消息队列
-------

MQ 全称为Message Queue, 消息队列。是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。

消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信。队列的使用除去了接收和发送应用程序同时执行的要求。
在项目中，将一些无需即时返回且耗时的操作提取出来，进行了异步处理，而这种异步处理的方式大大的节省了服务器的请求响应时间，从而提高了系统的吞吐量。

RabbitMQ 应用场景
---------------

对于一个大型的软件系统来说，它会有很多的组件或者说模块或者说子系统或者（subsystem or Component or submodule）。那么这些模块的如何通信？这和传统的IPC有很大的区别。传统的IPC很多都是在单一系统上的，模块耦合性很大，不适合扩展（Scalability）；如果使用socket那么不同的模块的确可以部署到不同的机器上，但是还是有很多问题需要解决。比如：

> 1）信息的发送者和接收者如何维持这个连接，如果一方的连接中断，这期间的数据如何防止丢失？
> 
> 2）如何降低发送者和接收者的耦合度？
> 
> 3）如何让Priority高的接收者先接到数据？
> 
> 4）如何做到load balance？有效均衡接收者的负载？
> 
> 5）如何有效的将数据发送到相关的接收者？也就是说将接收者subscribe 不同的数据，如何做有效的filter。
> 
> 6）如何做到可扩展，甚至将这个通信模块发到cluster上？
> 
> 7）如何保证接收者接收到了完整，正确的数据？

概念介绍
-------

- Broker：简单来说就是消息队列服务器实体。
- Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
- Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
- Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
- Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
- vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
- producer：消息生产者，就是投递消息的程序。
- consumer：消息消费者，就是接受消息的程序。
- channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

![Rabbitmq](https://upload-images.jianshu.io/upload_images/683118-b22270d646a5aeef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/691/format/webp)

RabbitMQ使用流程
--------------

AMQP模型中，消息在producer中产生，发送到MQ的exchange上，exchange根据配置的路由方式发到相应的Queue上，Queue又将消息发送给consumer，消息从queue到consumer有push和pull两种方式。 消息队列的使用过程大概如下：

1. 客户端连接到消息队列服务器，打开一个channel。
2. 客户端声明一个exchange，并设置相关属性。
3. 客户端声明一个queue，并设置相关属性。
4. 客户端使用routing key，在exchange和queue之间建立好绑定关系。
5. 客户端投递消息到exchange。

exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。 exchange也有几个类型，完全根据key进行投递的叫做Direct交换机，例如，绑定时设置了routing key为”abc”，那么客户端提交的消息，只有设置了key为”abc”的才会投递到队列。

**RabbitMQ的安装** : 查看下一篇文章

Java入门实例(Helloworld)
---------------------

一个producer发送消息，一个接收者接收消息，并在控制台打印出来。

![mq消费过程](https://upload-images.jianshu.io/upload_images/683118-71798beda1e40579.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/392/format/webp)

**Java客户端配置**

加入pom依赖

```xml
<dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.0.0</version>
</dependency>
```
**发送端**：Send.java 连接到RabbitMQ（此时服务需要启动），发送一条数据，然后退出。

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.concurrent.TimeoutException;

public class Send {
    //队列名称
    private final static String QUEUE_NAME = "helloMQ";

    public static void main(String[] argv) throws java.io.IOException, TimeoutException {
        /**
         * 创建连接连接到MabbitMQ
         */
        ConnectionFactory factory = new ConnectionFactory();
        //设置MabbitMQ所在主机ip或者主机名
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("123456");
        //创建一个连接
        Connection connection = factory.newConnection();
        //创建一个频道
        Channel channel = connection.createChannel();
        //指定一个队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //发送的消息
        String message = "hello world!";
        //往队列中发出一条消息
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println(" [x] Sent '" + message + "'");
        //关闭频道和连接
        channel.close();
        connection.close();
    }
}
```

值得注意的是队列只会在它不存在的时候创建，多次声明并不会重复创建。信息的内容是字节数组，也就意味着你可以传递任何数据。

**接收端**：Recv.java 不断等待服务器推送消息，然后在控制台输出。

```java
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv {
    // 队列名称
    private final static String QUEUE_NAME = "helloMQ";

    public static void main(String[] argv) throws Exception {
        // 打开连接和创建频道，与发送端一样
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明队列，主要为了防止消息接收者先运行此程序，队列还不存在时创建队列。
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        //创建消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
            }
        };
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

RabbitMQ工作队列-Work Queues（Java实例）
-----------------------------------

创建一个工作队列用来在工作者（consumer）间分发耗时任务。

![RabbitMQ工作队列](https://upload-images.jianshu.io/upload_images/683118-1153d8d6c2a8d9a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/332/format/webp)

工作队列的主要任务是：避免立刻执行资源密集型任务，然后必须等待其完成。相反地，我们进行任务调度：我们把任务封装为消息发送给队列。工作进行在后台运行并不断的从队列中取出任务然后执行。当你运行了多个工作进程时，任务队列中的任务将会被工作进程共享执行。

我们使用Thread.sleep来模拟耗时的任务, 然后启动两个work, 可以发现`消费时间变短`了

**发送端**

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class NewTask {
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        String message = String.valueOf(System.currentTimeMillis());
        channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));

        System.out.println(" [x] Sent '" + message + "'");
        channel.close();
        connection.close();
    }
}
```

**接收端**

```java
import com.rabbitmq.client.*;

import java.io.IOException;

public class Worker {
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("123456");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
        channel.basicQos(1);

        final Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
                try {
                    try {
                        Thread.sleep( 5000);    // 执行耗时操作
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } finally {
                    System.out.println(" [x] Done" + message);
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        channel.basicConsume(TASK_QUEUE_NAME, false, consumer);
    }
}
```
使用任务队列的好处是能够很容易的并行工作。如果我们积压了很多工作，我们仅仅通过增加更多的工作者就可以解决问题，使系统的伸缩性更加容易。

**消息确认**
执行一个任务需要花费几秒钟。你可能会担心当一个工作者在执行任务时发生中断。我们上面的代码，一旦RabbItMQ交付了一个信息给消费者，会马上从内存中移除这个信息。在这种情况下，如果杀死正在执行任务的某个工作者，我们会丢失它正在处理的信息。我们也会丢失已经转发给这个工作者且它还未执行的消息。如下:

```java
boolean ack = false ; //打开消息应答机制  
channel.basicConsume(QUEUE_NAME, ack, consumer);  
//另外需要在每次处理完成一个消息后，手动发送一次应答。  
channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);  
```
**消息的持久性**

虽然消费者被杀死，消息也不会被丢失。但是如果此时RabbitMQ服务被停止，我们的消息仍然会丢失。
当RabbitMQ退出或者异常退出，将会丢失所有的队列和信息，除非你告诉它不要丢失。我们需要做两件事来确保信息不会被丢失：我们需要给所有的队列和消息设置持久化的标志。

第一， 我们需要确认RabbitMQ永远不会丢失我们的队列。为了这样，我们需要声明它为持久化的。

```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

注：RabbitMQ不允许使用不同的参数重新定义一个队列，所以已经存在的队列，我们无法修改其属性。

第二， 我们需要标识我们的信息为持久化的。通过设置MessageProperties（implements BasicProperties）值为`PERSISTENT_TEXT_PLAIN`

```java
channel.basicPublish("", "task_queue",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
```
现在你可以执行一个发送消息的程序，然后关闭服务，再重新启动服务，运行消费者程序测试。

**公平的分配**

或许会发现，目前的消息转发机制（Round-robin）并非是我们想要的。例如，这样一种情况，对于两个消费者，有一系列的任务，奇数任务特别耗时，而偶数任务却很轻松，这样造成一个消费者一直繁忙，另一个消费者却很快执行完任务后等待。
造成这样的原因是因为RabbitMQ仅仅是当消息到达队列进行转发消息。并不在乎有多少任务消费者并未传递一个应答给RabbitMQ。仅仅盲目转发所有的奇数给一个消费者，偶数给另一个消费者。

为了解决这样的问题，我们可以使用basicQos方法，传递参数为prefetchCount = 1。这样告诉RabbitMQ不要在同一时间给一个消费者超过一条消息。换句话说，只有在消费者空闲的时候会发送下一条信息。

```java
int prefetchCount = 1;  
channel.basicQos(prefetchCount);  
```