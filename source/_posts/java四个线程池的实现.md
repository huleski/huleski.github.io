---
title: java四个线程池的实现
categories: java
tags: java
date: 2019-03-11 20:13:02
---

太久没有关注线程池, 现在复习一下

Java通过Executors提供四种线程池，分别为：
```
newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
```

newCachedThreadPool
------------------

创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下：
``` 
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();  
        for (int i = 0; i < 10; i++) {  
            final int index = i;  
            try {  
                Thread.sleep(index * 1000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            cachedThreadPool.execute(new Runnable() {  
                public void run() {  
                    System.out.println(index);  
                }  
            });  
        }  
    }  
}
```
线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

newFixedThreadPool
---------------

创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。示例代码如下：
```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ExecutorService fixedThreadPool  = Executors.newFixedThreadPool(3);  
        for (int i = 0; i < 10; i++) {  
            fixedThreadPool.execute(new Runnable() {  
                public void run() {  
                    try {  
                        System.out.println(index);  
                        Thread.sleep(2000);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
        }  
    }  
}
```
因为线程池大小为3，每个任务输出index后sleep 2秒，所以每两秒打印3个数字。
定长线程池的大小最好根据系统资源进行设置。如Runtime.getRuntime().availableProcessors()

newScheduledThreadPool
--------------------

创建一个定长线程池，支持定时及周期性任务执行。延迟执行示例代码如下：
```
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);  
        scheduledThreadPool.schedule(new Runnable() {  
            public void run() {  
                System.out.println("delay 3 seconds");  
            }  
        }, 3, TimeUnit.SECONDS);  
    }  
}  
```
表示延迟3秒执行。

定期执行示例代码如下：
```
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);  
        scheduledThreadPool.scheduleAtFixedRate(new Runnable() {  
            public void run() {  
                System.out.println("delay 1 seconds, and excute every 3 seconds");  
            }  
        }, 1, 3, TimeUnit.SECONDS);  
    }  
}  
```
表示延迟1秒后每3秒执行一次。

newSingleThreadExecutor
----------------------
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。示例代码如下：
```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 10; i++) {  
            final int index = i;  
            singleThreadExecutor.execute(new Runnable() {  
                public void run() {  
                    try {  
                        System.out.println(index);  
                        Thread.sleep(2000);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
        }  
    }  
}  
```
结果依次输出，相当于顺序执行各个任务。


可以使用JDK自带的监控工具来监控我们创建的线程数量，运行一个不终止的线程，创建指定量的线程，来观察：

工具目录：C:\Program Files\Java\jdk1.6.0_06\bin\jconsole.exe

运行程序做稍微修改：
```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  

public class ThreadPoolExecutorTest {  
    public static void main(String[] args) {  
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();  
        for (int i = 0; i < 100; i++) {  
            final int index = i;  
            cachedThreadPool.execute(new Runnable() {  
                public void run() {  
                    try {  
                        while(true) {  
                            System.out.println(index);  
                            Thread.sleep(10 * 1000);  
                        }  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
            try {  
                Thread.sleep(500);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
} 
```
可以查看线程数的变化
