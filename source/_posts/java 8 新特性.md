---
title: java 8 本新特性
categories: java
tags: java
date: 2019-11-12 20:17:25
---

[官网介绍](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)

参考: [【译】Java 8的新特性—终极版](https://www.jianshu.com/p/5b800057f2d8)

- Lambda 表达式 和 函数式接口 − Lambda 允许把函数作为一个方法的参数（函数作为参数传递到方法中）。
==========

```
// Function<T, R> -T作为输入， 返回的R作为输出
Function<String,String> fun = (x) -> {System.out.print(x+": ");return "Function";};
System.out.println(function.apply("hello world"));

//Predicate<T> -T作为输入， 返回的boolean值作为输出
Predicate<String> pre = (x) ->{System.out.print(x);return false;};
System.out.println(": "+pre.test("hello World"));

//Consumer<T> - T作为输入， 执行某种动作但没有返回值
Consumer<String> con = (x) -> {System.out.println(x);};
con.accept("hello world");

//Supplier<T> - 没有任何输入， 返回T
Supplier<String> supp = () -> {return "Supplier";};
System.out.println(supp.get());

//BinaryOperator<T> -两个T作为输入， 返回一个T作为输出， 对于“reduce”操作很有用
BinaryOperator<String> bina = (x,y) ->{System.out.print(x+" "+y);return "Binary Operator";};
System.out.println(" "+bina.apply("hello ","world"))
```

- 方法引用 − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

- 默认方法 − 默认方法就是一个在接口里面有了一个实现的方法。

- 应用范围扩大的注解, 可以重复注解, 更好的类型推断

- 新工具 − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。

- Stream API −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。

流的特点
____________

1. 只能遍历一次

 我们可以把流想象成一条流水线， 流水线的源头是我们的数据源(一个集合)， 数据源中的元素依次被输送到流水线上， 我们可以在流水线上对元素进行各种操作。

一旦元素走到了流水线的另一头， 那么这些元素就被“消费掉了” ， 我们无法再对这个流进行操作。 当然，
我们可以从数据源那里再获得一个新的流重新遍历一遍。
 
2. 采用内部迭代方式
   
若要对集合进行处理， 则需我们手写处理代码， 这就叫做外部迭代。

而要对流进行处理， 我们只需告诉流我们需要什么结果， 处理过程由流自行完成， 这就称为内部迭代。

流的操作种类
__________

流的操作分为两种， 分别为中间操作和终端操作。

1. 中间操作

当数据源中的数据上了流水线后， 这个过程对数据进行的所有操作都称为“中间操作” 。
中间操作仍然会返回一个流对象， 因此多个中间操作可以串连起来形成一个流水线。

2. 终端操作

当所有的中间操作完成后， 若要将数据从流水线上拿下来， 则需要执行终端操作。
终端操作将返回一个执行结果， 这就是你想要的数据

List 转 Stream
____________

```
// 转stream
list.stream()
// 并发处理
list.parallelStream()
```

filter（ 过滤）
___________

```
Stream<T> filter(Predicate<? super T> predicate);
```

map（ 元素转换）
_____________

```
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
IntStream mapToInt(ToIntFunction<? super T> mapper);
LongStream mapToLong(ToLongFunction<? super T> mapper);
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
```

flatMap（ 元素转换）
____________

```
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper); IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);
LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);
DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);
```

distinct（ 去除重复， 对象需要重写 equals、 hashCode）
_____________________

```
Stream<T> distinct();
```

sorted（ 排序）
_____________

```
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

peek（ 生成新的流： 流是单向的， 例如用于日志打印）
____________

```
Stream<T> peek(Consumer<? super T> action);
```

limit（ 取前面 n 个元素）
______________

```
Stream<T> limit(long maxSize);
```

skip（ 跳过 n 个元素）
______________

```
Stream<T> skip(long n);
```

forEach（ 遍历）
_____________

```
void forEach(Consumer<? super T> action);
void forEachOrdered(Consumer<? super T> action);
```

toArray（ 转换成数组）
__________

```
Object[] toArray();
<A> A[] toArray(IntFunction<A[]> generator);
```

reduce（ 结果归并）
_____________

```
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity, 
            BiFunction<U, ? super T, U> accumulator,
            BinaryOperator<U> combiner);
```

collect（ 转换成集合）
_______________

```
<R> R collect(Supplier<R> supplier,
                BiConsumer<R, ? super T> accumulator,
                BiConsumer<R, R> combiner);
<R, A> R collect(Collector<? super T, A, R> collector);
```

转list
_______

```
// 转list
Collectors.toList();
// 转set
Collectors.toSet();
// 转map
List<TestVo> testList = new ArrayList<>(10);
Map<Long, TestVo> data = releaseList.stream()
            .collect(Collectors.toMap(TestVo::getId, x -> x));
```

count（ 计数）
_________

```
long count();
```

查找
_______

```
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
Optional<T> findFirst();
Optional<T> findAny();
```

- Date Time API − 加强对日期与时间的处理。

时区类 java.time.ZoneId
_______

```
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
ZoneId systemZoneId = ZoneId.systemDefault();
```

Instant类在Java日期与时间功能中， 表示了时间线上一个确切的点， 定义为距离初始时间的时间差
_______

```
// 一个Instant对象里有两个域： 距离初始时间的秒钟数、 在当前一秒内的第几纳秒
Instant now = Instant.now();
long seconds = systemZoneId.getEpochSecond();
int nanos = systemZoneId.getNano();

// 计算
Instant later = now.plusSeconds(3);
Instant earlier = now.minusSeconds(3);
```

Clock类提供了访问当前日期和时间的方法， Clock是时区敏感的
_______

```
// 可以用来取代System.currentTimeMillis() 来获取当前的微秒数。
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();
Instant instant = clock.instant();
Date legacyDate = Date.from(instant);
```

LocalDate类是Java 8中日期时间功能里表示一个本地日期的类， 它的日期是无时区属性的。
_______
```
LocalDate localDate = LocalDate.now();
LocalDate localDate2 = LocalDate.of(2018, 3, 3);

// 可以用如下方法访问LocalDate中的日期信息
int year = localDate.getYear();
Month month = localDate.getMonth();
int dayOfMonth = localDate.getDayOfMonth();
int dayOfYear = localDate.getDayOfYear();
DayOfWeek dayOfWeek = localDate.getDayOfWeek();

// 计算
LocalDate d1 = localDate.plusYears(3);
LocalDate d2 = localDate.minusYears(3);
```

LocalTime类是Java 8中日期时间功能里表示一整天中某个时间点的类， 它的时间是无时区属性的
_______

```
LocalTime localTime = LocalTime.now();
LocalTime localTime2 = LocalTime.of(21, 30, 59, 11001);
// 通过这些方法访问其时、 分、 秒、 纳秒
getHour()
getMinute()
getSecond()
getNano()
// 计算
LocalTime localTimeLater = localTime.plusHours(3);
LocalTime localTimeEarlier = localTime.minusHours(3);
```

LocalDateTime类是Java8中无时区的日期时间, 相当于LocalDate与LocalTime两个类的结合，使用方法也类似
_______

```
LocalDateTime localDateTime = LocalDateTime.now();
······
```

ZonedDateTime类是Java 8中日期时间功能里， 用于表示带时区的日期与时间信息的类
_______

```
ZonedDateTime dateTime = ZonedDateTime.now();
ZoneId zoneId = ZoneId.of("UTC+1");
ZonedDateTime dateTime2 = ZonedDateTime.of(2015, 11, 30, 23, 45, 59, 1234, zoneId);

// 方法使用,计算同上
······
```

DateTimeFormatter类是Java 8中日期时间功能里， 用于解析和格式化日期时间的类
_______

```
// 类中包含如下预定义的实例
BASIC_ISO_DATE
ISO_LOCAL_DATE
ISO_LOCAL_TIME
ISO_LOCAL_DATE_TIME
······

// 格式化为某种字符串
DateTimeFormatter formatter = DateTimeFormatter.BASIC_ISO_DATE;
String formattedDate = formatter.format(LocalDate.now());
```

Duration对象表示两个Instant间的一段时间
______________

```
Instant first = Instant.now();
// wait some time while something happens
Instant second = Instant.now();
Duration duration = Duration.between(first, second);

// 一个Duration对象里有两个域： 纳秒值（ 小于一秒的部分） ， 秒钟值（ 一共有几秒） 
long seconds = getSeconds()
int nanos = getNano()

// 转换
long days = duration.toDays(); // 这段时间的总天数
long hours = duration.toHours(); // 这段时间的小时数
long minutes = duration.toMinutes(); // 这段时间的分钟数
long seconds = duration.getSeconds(); // 这段时间的秒数

// 计算
Duration start = ... //obtain a start duration
Duration added = start.plusDays(3);
Duration subtracted = start.minusDays(3)
```

TemporalAdjuster类可以更方便的调整时间
————————————————————————————

```
LocalDateTime localDateTime = LocalDateTime.now();
// 返回下一个距离当前时间最近的星期六
LocalDateTime dateTime1 = localDateTime.with(TemporalAdjusters.nextOrSame(DayOfWeek.SATURDAY));
//  返回本月最后一个星期五
LocalDateTime dateTime2 = localDateTime.with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY));
```

- Optional 类 − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。

```
// 参数不能是null
Optional optional1 = Optional.of(1);
// 参数可以是null
Optional optional2 = Optional.ofNullable(null);
// 参数可以是非null
Optional optional3 = Optional.ofNullable(2);

// isPresent判断值是否存在
System.out.println(optional1.isPresent() == true);
System.out.println(optional2.isPresent() == false);

// orElse
System.out.println(optional1.orElse(1000) == 1);// true
System.out.println(optional2.orElse(1000) == 1000);// true
```

- Nashorn, JavaScript 引擎 − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。

- JVM的新特性 - 使用Metaspace（JEP 122）代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize。
