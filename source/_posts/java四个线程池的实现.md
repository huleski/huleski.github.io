---
title: java四个线程池的实现
categories: java
tags: java
date: 2019-03-11 20:13:02
---

多线程的几种实现方式
-----------------

**1、实现Runnable接口**
```
public class DemoThreadTask implements Runnable{
    @Override
    public void run() {
        // TODO Auto-generated method stub
    }
    
    public static void main(String[] args) {
        DemoThreadTask task = new DemoThreadTask();
        Thread t = new Thread(task);
        t.start();
        ...
    }
}
```

实现Runnable接口，利用Runnable实例构造Thread，是较常用且最本质实现。此构造方法相当于对Runnable实例进行一层包装，在线程t启动时，调用Thread的run方法从而间接调用target.run()：
```
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;

    public void run() {
        if (target != null) {
            target.run();
        }
   }
     ...
}
```

**2、继承Thread类**
```
public class DemoThread extends Thread{
    @Override 
    //重写run方法
    public void run() {
        // TODO Auto-generated method stub
    }

    public static void main(String[] args) {
        DemoThread t = new DemoThread();
        t.start();
        ...
    }
}
```

这种实现方式是显示的继承了Thread，但从类图中我们可以看到，Thread类本身就继承自Runnable，所以继承Thread的本质依然是实现Runnable接口定义的run方法。

需要注意的是继承Thread方式，target对象为null，重写了run方法，导致方式1中的Thread原生的run方法失效，因此并不会调用到target.run()的逻辑，而是直接调用子类重写的run方法。

因为java是单根继承，此方式一般不常用。

**3、实现Callable接口并通过FutureTask包装**
```
public class DemoCallable implements Callable<String>{
    @Override
    public String call() throws Exception {
        // TODO Auto-generated method stub
        return null;
    }
    
    public static void main(String[] args) throws Exception {
        DemoCallable c = new DemoCallable();
        FutureTask<String> future = new FutureTask<>(c); 
        Thread t = new Thread(future);
        t.start();
        ...
        String result = future.get(); //同步获取返回结果
        System.out.println(result);
    }
}
```

实现Callable接口通过FutureTask包装，可以获取到线程的处理结果，future.get()方法获取返回值，如果线程还没执行完，则会阻塞。

FutureTask实现了RunnableFuture，RunnableFuture则实现了Runnable和Future两个接口。因此构造Thread时，FutureTask还是被转型为Runnable使用。因此其本质还是实现Runnable接口。

**4、匿名内部类**

匿名内部类也有多种变体，上述三种方式都可以使用匿名内部类来隐式实例化。
```
public class Demo{
    
    public static void main(String[] args) throws Exception {
        //方式一：Thread匿名内部类
        new Thread(){
            @Override
            public void run() {
                // TODO Auto-generated method stub
            }
        }.start();
        
        //方式二：Runnable匿名内部类
        new Thread(new Runnable() {
            @Override
            public void run() {
                // TODO Auto-generated method stub
            }
        }).start();
        ...
    }
}
```

**5、Lambda表达式**

```
public class Demo{
    public static void main(String[] args) throws Exception {
        new Thread(() -> System.out.println("running") ).start() ;
        ...
    }
}
```

**6、线程池**

```
public class DemoThreadTask implements Runnable{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        System.out.println("running");
    }
    
    public static void main(String[] args) {
        DemoThreadTask task = new DemoThreadTask();
        ExecutorService ex = Executors.newCachedThreadPool();
        ex.execute(task);
        ...
    }
}
```

线程池与前面所述其他方式的区别在于执行线程的时候由ExecutorService去执行，最终还是利用Thread创建线程。线程池的优势在于线程的复用，从而提高效率。

**7、定时器**

```
public class DemoTimmerTask {

    public static void main(String[] args) throws Exception {
        Timer timer = new Timer();
        timer.scheduleAtFixedRate((new TimerTask() {
            @Override
            public void run() {
                System.out.println("定时任务1执行了....");
            }
        }), 2000, 1000);
    }
}
```

TimerTask的实现了Runnable接口，Timer内部有个TimerThread继承自Thread，因此绕回来还是Thread + Runnable。


Java通过Executors提供四种线程池
-------------------------

```
newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
```

**1. newCachedThreadPool**

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

**2. newFixedThreadPool**

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

**3. newScheduledThreadPool**

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

**4. newSingleThreadExecutor**

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
