---
title: Java虚拟机GC详解
categories: java
tags: java
date: 2019-04-13 18:39:00
---

## 1. GC的定义和价值

​	在C/C++里是由程序猿自己去申请、管理和释放内存空间，因此没有GC的概念。而在Java中，后台专门有一个专门用于垃圾回收的线程来进行监控、扫描，自动将一些无用的内存进行释放，这就是垃圾收集的一个基本思想，目的在于防止人为的内存泄露。

　	Java GC（Garbage Collection，垃圾收集，垃圾回收）机制，是Java与C++/C的主要区别之一，作为Java开发者，一般不需要专门编写内存回收和垃圾清理代 码，对内存泄露和溢出的问题，也不需要像C程序员那样战战兢兢。这是因为在Java虚拟机中，存在自动内存管理和垃圾清扫机制。

​	概括地说，该机制对 JVM（Java Virtual Machine）中的内存进行标记，并确定哪些内存需要回收，根据一定的回收策略，自动的回收内存，永不停息的保证JVM中的内存空间，防止出现内存泄露和溢出问题。



## 2. 思考GC的运行原理

```
第一步：确认那些对象需要回收
第二步：使用什么方法回收
```



## 3. 确认哪些对象需要回收


常见的算法有2种：引用计数算法和根搜索算法。
* 引用计算法无法解决循环引用问题，java不采用，采用了根搜索算法。




#### 3-1 引用计数法

###### 3-1-1 概念

```s
给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的。
```

##### 3-1-2 特点


引用计数算法的实现简单，判定效率也高，大部分情况下是一个不错的算法。
它很难解决对象之间相互循环引用的问题,对于循环引用的对象无法进行回收

* 由于循环引用的计数器都不为0，但是他们对于根对象都已经不可达了，但是无法释放。


#### 3-2 根搜索算法

#### 3-2-1 概念

```
由于引用计数算法的缺陷，所以JVM一般会采用一种新的算法，叫做根搜索算法。它的处理方式就是，设立若干种根对象，当任何一个根对象到某一个对象均不可达时，则认为这个对象是可以被回收的。
```

![Logo](/images/jvm_gc/jvm_gc1.webp)

#### 3-2-2 特点

```
如上图所示, 由于GC roots到灰色对象部分不可达，所以最终灰色对象部分还是会被当做GC的对象，上图若是采用引用计数法，则灰色对象部分都不会被回收。
```


3-2-3 可达性的解释

```
1. 来历
* 我们刚刚提到，设立若干种根对象，当任何一个根对象到某一个对象均不可达时，则认为这个对象是可以被回收的。
* 我们在后面介绍标记-清理算法/标记整理算法时，也会一直强调从根节点开始，对所有可达对象做一次标记，那什么叫做可达呢？

2. 讲解
* 这里解释如下：可达性分析：从根（GC Roots）的对象作为起始点，开始向下搜索，搜索所走过的路径称为“引用链”，当一个对象到GC Roots没有任何引用链相连（用图论的概念来讲，就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的。
	
3. jvm常见的根(GC roots)对象
	a. 栈（栈帧中的本地变量表）中引用的对象。
	b. 方法区中的静态成员。
	c. 方法区中的常量引用的对象（全局变量
	d. 方法栈中JNI（一般说的Native方法）引用的对象。
	
	**注：第一和第四种都是指的方法的本地变量表，第二种表达的意思比较清晰，第三种主要指的是声明为final的常量值。
```


## 4. 基础的GC回收算法

```
* 在根搜索算法的基础上，现代虚拟机的实现当中，垃圾搜集的算法主要有三种，分别是标记-清除算法、复制算法、标记-整理算法。
```



#### 4-1  最基础：标记/清除算法

```
1. 介绍
标记/清除算法是几种GC算法中最基础的算法，是因为后续的收集算法都是基于这种思路并对其不足进行改进而得到的。
	
2. 原理
* 标记/清除算法的基本思想就跟它的名字一样，分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。
	
* 标记阶段：标记的过程其实就是前面介绍的可达性分析算法的过程，遍历所有的GC Roots对象，对从GC Roots对象可达的对象都打上一个标识，一般是在对象的header中，将其记录为可达对象；
	
* 清除阶段：清除的过程是对堆内存进行遍历，如果发现某个对象没有被标记为可达对象（通过读取对象header信息），则将其回收。
	
3. 具体的算法过程
* 在标记阶段，从对象GC Root 1可以访问到B对象，从B对象又可以访问到E对象，因此从GC Root 1到B、E都是可达的，同理，对象F、G、J、K都是可达对象；到了清除阶段，所有不可达对象都会被回收。
	
* 在垃圾收集器进行GC时，必须停止所有Java执行线程（也称"STW, Stop The World"），原因是在标记阶段进行可达性分析时，不可以出现分析过程中对象引用关系还在不断变化的情况，否则的话可达性分析结果的准确性就无法得到保证。在等待标记清除结束后，应用线程才会恢复运行。
```


```
存在的缺陷:
1、效率问题。标记和清除两个阶段的效率都不高，因为这两个阶段都需要遍历内存中的对象，很多时候内存中的对象实例数量是非常庞大的，这无疑很耗费时间，而且GC时需要停止应用程序，这会导致非常差的用户体验。
2、空间问题。标记清除之后会产生大量不连续的内存碎片（从上图可以看出），内存空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾回收动作。
	
* 内存碎片的影响
```



#### 4-2 复制算法

```
1. 原理
* 复制算法为了解决效率问题，复制算法出现了。复制算法的原理是：将可用内存按容量划分为大小相等的两块，每次使用其中的一块。当这一块的内存用完了，就将还存活的对象复制到另一块内存上，然后把这一块内存所有的对象一次性清理掉。
	
2. 具体的算法过程
* 复制算法每次都是对整个半区进行内存回收，这样就减少了标记对象遍历的时间，在清除使用区域对象时，不用进行遍历，直接清空整个区域内存，而且在将存活对象复制到保留区域时也是按地址顺序存储的，这样就解决了内存碎片的问题，在分配对象内存时不用考虑内存碎片等复杂问题，只需要按顺序分配内存即可。
```


```
1. 优缺点
* 复制算法简单高效，优化了标记/清除算法的效率低、内存碎片多的问题。
* 缺点也很明显：
	a. 将内存缩小为原来的一半，浪费了一半的内存空间，代价太高
	b. 如果对象的存活率很高，极端一点的情况假设对象存活率为100%，那么我们需要将所有存活的对象复制一遍，耗费的时间代价也是不可忽视的。
```

#### 4-3 标记/整理算法

```
1. 原理
    从名字上看，这种算法与标记/清除算法很像，事实上，标记/整理算法的标记过程任然与标记/清除算法一样，但后续步骤不是直接对可回收对象进行回收，而是让所有存活的对象都向一端移动，然后直接清理掉端边线以外的内存。
	
2. 算法分析
    回收后可回收对象被清理掉了，存活的对象按规则排列存放在内存中。这样一来，当我们给新对象分配内存时，jvm只需要持有内存的起始地址即可。标记/整理算法不仅弥补了标记/清除算法存在内存碎片的问题，也消除了复制算法内存减半的高额代价，可谓一举两得。但任何算法都有缺点，就像人无完人，标记/整理算法的缺点就是效率也不高，不仅要标记存活对象，还要整理所有存活对象的引用地址，在效率上不如复制算法。
```


#### 4-4 总结

```
* 弄清了以上三种算法的原理，下面我们来从几个方面对这几种算法做一个简单排行。

效率：复制算法 > 标记/整理算法 > 标记/清除算法（标记/清除算法有内存碎片问题，给大对象分配内存时可能会触发新一轮垃圾回收）
内存整齐率：复制算法 = 标记/整理算法 > 标记/清除算法	
内存利用率：标记/整理算法 = 标记/清除算法 > 复制算法

从上面简单的评估可以看出，标记/清除算法已经比较落后了，但是吃水不忘挖井人，它是后面几种算法的前辈、是基础，在某些场景下它也有用武之地。
```


## 5.  JVM采用的GC回收算法：分代回收算法

#### 5-1 概念

```
1. 引言
* 通过上面的分析，每种算法都有各自的特点，没有完美的解决方案。所以，JVM虚拟机，根据自身的特点，设计了一个特别的GC回收机制： 分代回收算法。

2. 分代回收算法概念
* 根据对象的存活周期的不同，将内存划分为几块儿。
* Java堆分为新生代和老年代：短命对象归为新生代，长命对象归为老年代。
```

#### 5-2 原理

```
分代回收算法原理
a. 少量对象存活，适合复制算法：
* 在新生代中，每次GC时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成GC。
	
b. 大量对象存活，适合用标记-清理/标记-整理：
* 在老年代中，因为对象存活率高、没有额外空间对他进行分配担保，就必须使用“标记-清理”/“标记-整理”算法进行GC。
```

#### 5-3 新生代和老年代对象的来历

```
a. 所有第一次分配的对象，都是新生代。
b. 新生代的内存满后，启动GC，大概98%的对象会被回收，2%的对象，会存活下来。每经历一次GC，对象的年龄增长1岁。
c. 当年龄达到阀值（默认是15，可设定），即经历了15次GC，仍然活着，就从新生代转移到老年代。

d. 老年代还有来历特别的对象：新生代进行垃圾回收时，某个对象特别大，可能无需等到15岁，就直接进入老年代
```

#### 5-3 新生代的回收策略

```
1. 新生代的特点
* 新生代中的对象几乎都是“朝生夕死”的（达到98%，即98%的对象活不过1岁）

2. 选择复制算法的原因分析
* 复制算法的效率最高，但是浪费50%的内存。
* 但是新生代的对象存活率低，所以并不需要按照1：1的比例来划分内存空间
* 而是将内存分为一块较大的Eden空间和两块较小的Survivor1空间、Survivor2空间，三者的比例为8：1：1。
	
3. 算法的实现
* 将内存分为一块比较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。
* 当回收时，将Eden和Survivor中还存活着的对象一次性地复制到另外一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。
* 默认Eden和Survivor的大小比例是8:1，也就是说，每次新生代中可用内存空间为整个新生代容量的90%（80%+10%），只有10%的空间会被浪费。
* 98%的对象可回收只是一般场景下的数据，我们没有办法保证每次回收都只有不多于10%的对象存活，当Survivor空间不够用时，需要依赖于老年代进行分配担保，所以大对象直接进入老年代。
```



#### 5-4 GC分类

```
Minor GC：只有新生代进行GC，发生频繁，但是，STW(stop the world)的时间短。
Major GC: 年长代进行GC, 因年长代空间比新生代大，故运行次数少，其一般采用“标记-清理”/“标记-整理”，STW的时间长。
FULL GC: 新生代和年老代，都进行GC操作。所需时间最长，STW最长。
```

#### 5-5 STW(Stop-The-World)

```
1. 概念
* Java中一种全局暂停的现象。全局停顿，所有Java代码停止，native代码可以执行，但不能和JVM交互多半情况下是由于GC引起。
	
2. GC引起STW的原因
* 打个比方：类比在聚会，突然GC要过来打扫房间，聚会时很乱，又有新的垃圾产生，房间永远打扫不干净，只有让大家停止活动了，才能将房间打扫干净。况且，如果没有全局停顿，会给GC线程造成很大的负担，GC算法的难度也会增加，GC很难去判断哪些是垃圾。
	
3. 危害
* 长时间服务停止，没有响应

```

JVM GC 垃圾回收器类型
-------------------

JVM的垃圾回收器大致分为四种类型：
![jvm类型](/images/jvm_gc/jvm_gc2.jpg)

**1、串行垃圾回收器  Serial Garbage Collector**

串行垃圾回收器在进行垃圾回收时，它会持有所有应用程序的线程，冻结所有应用程序线程，使用单个垃圾回收线程来进行垃圾回收工作。
串行垃圾回收器是为单线程环境而设计的，如果你的程序不需要多线程，启动串行垃圾回收。（一般是command line程序）
使用方法：-XX:+UseSerialGC 
Ps：在jdk client模式，不指定VM参数，默认是串行垃圾回收器

![串行垃圾回收器](/images/jvm_gc/jvm_gc3.jpg)

**2、并行垃圾回收器  Parallel Garbage Collector**

并行垃圾回收器在进行垃圾回收时，同样会持有所有应用程序的线程，并冻结所有应用程序线程，来进行垃圾回收工作。
唯一和串行垃圾回收器不同的是，并行垃圾回收器是使用多线程来进行垃圾回收工作的。

![并行垃圾回收器](/images/jvm_gc/jvm_gc4.jpg)

**3、并发标记扫描垃圾回收器 CMS Garbage Collector**

Concurrent Mark Sweep (CMS)垃圾回收器使用并发标记算法，使用多线程来扫描heap memory来标记实例，然后清理被标记过的实例。
CMS垃圾回收器有时候会Hold所有的应用程序线程，但有时候只会Hold部分应用程序线程。

如果能分配更多的CPU给垃圾回收器，那么CMS会是一个比并行垃圾回收更好的选择。XX:+USeParNewGC

![并发标记扫描垃圾回收器](/images/jvm_gc/jvm_gc5.jpg)

**4、G1垃圾回收器  G1 Garbage Collector**

G1垃圾回收器是用在heap memory很大的情况下，把heap划分为很多很多的region块，然后并行的对其进行垃圾回收。
G1垃圾回收器在清除实例所占用的内存空间后，还会做内存压缩。

G1垃圾回收器回收region的时候基本不会STW，而是基于 most garbage优先回收 的策略来对region进行垃圾回收的。

–XX:+UseG1GC

java8中，使用-XX:+UseStringDeduplication。这个优化会优化冗余的string为一个char数组。

![G1垃圾回收器](/images/jvm_gc/jvm_gc6.jpg)


查看JVM使用的默认的垃圾收集器

```bash
java -XX:+PrintCommandLineFlags -version
```

jvm配置
------

Option	|	Description
--------|-------------
-XX:+UseSerialGC	|	Serial Garbage Collector 串行垃圾回收器
-XX:+UseParallelGC	|	Parallel Garbage Collector并行垃圾回收器
-XX:+UseConcMarkSweepGC	|	CMS Garbage Collector并发标记垃圾回收器
-XX:ParallelCMSThreads=	|	CMS Collector – number of threads to use 并发标记垃圾回收器使用的线程数，通常是cpu个数
-XX:+UseG1GC	|	G1 Gargbage Collector 使用G1垃圾回收器

优化选项

Option	|	Description
-------|--------------
-Xms	|	Initial heap memory size 初始化heap大小 -Xms512M
-Xmx	|	Maximum heap memory size 设置最大的heap大小
-Xmn	|	Size of Young Generation 年轻代的大小
-XX:PermSize	|	Initial Permanent Generation size 初始化永久带的大小
-XX:MaxPermSize	|	Maximum Permanent Generation size 最大的永久带大小

总结
----

垃圾回收器目前分为四种类型, 串行，并行，并发标记，G1。

小数据量和小型应用，使用串行垃圾回收器即可。

对于对响应时间无特殊要求的，可以使用并行垃圾回收器和并发标记垃圾回收器。（中大型应用）

对于heap可以分配很大的中大型应用，使用G1垃圾回收器比较好，进一步优化和减少了GC暂停时间。

没有银弹，针对不同的场景，选用不同的垃圾回收器。