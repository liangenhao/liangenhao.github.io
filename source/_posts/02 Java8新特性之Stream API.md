---
title: 02 Java8新特性之Stream API
date: 2018-04-22
categories: Java新特性
tags: [Java8新特性, Lambda 表达式, Stream]
---

## 1 什么是Stream

> Java8的Stream与`java.io`包中的`InputStream`和`OutputStream`是完全不同的概念。它是对集合对象功能的增强。

Stream是对数据的操作，对数据操作需要有数据源（集合、数组），把对数据源的操作想象成对数据的传输，在传输中对数据源做一系列流水线式的中间操作。做完这些操作后，会产生一个新流，**原来的数据源是不会发生改变的**。

流（Stream）是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。

> 集合讲的是数据，流讲的是计算。

注意：

1. Stream 自己不会存储元素。
2. Stream 不会改变源对象。相反，它们会返回一个持有结果的新Stream。
3. Stream 操作是延迟执行的，这意味着它们会等到需要结果的时候才执行。

<!-- more -->

## 2 Stream 的操作三个步骤

1. 创建Stream

   一个数据源（集合、数组等），获取一个流。

2. 中间操作

   一个中间操作链，对数据源的数据进行处理。

3. 终止操作（终端操作）

   一个终止操作，执行中间操作链，并产生结果。

### 2.1 步骤一：创建 Stream

一、【方式一】：通过 Collection 系列集合提供的`stream()`或`parallelStream()`方法获取流。

> `stream()`获取的是串行流；`parallelStream`获取的是并行流。（串行流和并行流的区别往下看。）

```java
List<String> list = new ArrayList<>();
Stream<String> stream1 = list.stream();
```

二、【方式二】：通过`Arrays`中的静态方法`stream()`方法，获取数据流。

```java
User[] users = new User[10];
Stream<User> stream2 = Arrays.stream(users);
```

三、【方式三】：通过`Stream`类中的静态方法`of()`方法，获取流。

```java
// <T> Stream<T> of(T... values)
Stream<String> stream3 = Stream.of("aa", "bb", "cc");
```

四、【方式四】：创建无限流

```java
// 迭代:<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) 
Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 2);
// 生成:<T> Stream<T> generate(Supplier<T> s)
Stream.generate(() -> Math.random());
```

### 2.2 步骤二：中间操作

#### 2.2.1 筛选和切片

`filter`：接受 Lambda ，从流中排除某些元素。

​	`Stream<T> filter(Predicate<? super T> predicate);`

`limit(n)`：截断流，使其元素不超过给定数量。

​	`Stream<T> limit(long maxSize);`

`skip(n)`：跳过元素，返回一个扔掉了前n个元素的流，若流中元素不足n个，则返回一个空流，一`limit(n)`互补。

​	`Stream<T> skip(long n);`

`distinct`：筛选，通过流所生成元素的`hashCode()`和`equals()`去除重复元素。

​	`Stream<T> distinct();`

示例：

有数据源：

```java
List<User> userList = Arrays.asList(
            new User("101", "张三", 18, 9999.99),
            new User("102", "李四", 59, 6666.66),
            new User("103", "王五", 28, 3333.33),
            new User("104", "赵六", 8, 7777.77),
            new User("105", "田七", 38, 5555.55)
    );
```

一、【`filter`】：

```java
@Test
public void test2() {
  // 中间操作
  Stream<User> stream = userList.stream()
                      .filter((e) -> {
                        System.out.println("Stream API 的中间操作");
                        return e.getAge() > 35;
                      });
  // 终止操作
  stream.forEach(System.out::println);
}
```

> 注意：中间操作是不会执行任何操作**，只有执行终止操作以后，所有的中间操作一次性执行全部。这个过程称为`惰性求值`或`延迟加载`。**

> 我们没有做迭代操作， 由Stream API 自己完成迭代操作，称作`内部迭代`。

二、【`limit`】：

```java
@Test
public void test3() {
  userList.stream()
        .filter((e) -> {
          System.out.println("短路");
          return e.getSalary() > 5000;
        })
        .limit(2)
        .forEach(System.out::println); // 终止操作
}
```

> `limit(n)`：一旦发现了n条满足条件的数据后，其他后续的迭代操作不再继续进行，提高了效率，称作`短路`。

三、【`skip(n)`】：

```java
@Test
public void test4() {
  userList.stream()
          .filter((e) -> e.getSalary() > 5000)
          .skip(2) // 跳过前2个
          .forEach(System.out::println);
}
```

四、【`distinct`】：

```java
@Test
public void test4() {
  userList.stream()
          .filter((e) -> e.getSalary() > 5000)
          .skip(2) // 跳过前2个
          .distinct()
          .forEach(System.out::println);
}
```

> **User 类必须重写`hashCode()`和`equals()`方法。**

#### 2.2.2 映射

 `map`：接受 Lambda ， **将元素转换成其他形式或者提取信息。**接受一个函数作为参数，该函数会被应用到每个元素上，**并将其映射成一个新元素。**

​	`<R> Stream<R> map(Function<? super T, ? extends R> mapper);`

`flatMap`：接受一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连成一个流。

​	`<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);`

示例：

一、【`map`】：

```java
@Test
public void test5() {
  List<String> list = Arrays.asList("aaa", "bbb", "ccc");

  list.stream()
    .map((str) -> str.toUpperCase())
    .forEach(System.out::println);
}
```

> 将集合中的元素全部转换成大写。

```java
userList.stream()
        .map(User::getName)
        .forEach(System.out::println);
```

> 提取集合中的元素中的姓名。

二、【`flatMap`】：

```java
public static Stream<Character> filterCharacter(String str) {
  List<Character> list = new ArrayList<>();

  for (Character ch : str.toCharArray()) {
    list.add(ch);
  }
  return list.stream();
}

@Test
public void test6() {
  List<String> list = Arrays.asList("aaa", "bbb", "ccc");
  Stream<Character> characterStream = list.stream()
    .flatMap(TestStreamAPi1::filterCharacter);
  characterStream.forEach(System.out::println);
}
```

> `map`和`flatMap`的区别：例如有3个流对象。使用`map`是将这三个流对象一起放到一个流对象中。使用`flatMap`是将这三个流对象中的元素一起放到一个流对象中。
>
> 有点类似于集合中的`add`和`addAll`方法。

#### 2.2.3 排序

`sorted()`：自然排序。（`Comparable`）

`sorted(Comparator com)`：定制排序。（`comparator`）

示例：

一、【`sorted()`】：

```java
@Test
public void test7() {
  List<String> list = Arrays.asList("ccc", "aaa", "ddd", "bbb");

  list.stream()
    .sorted()
    .forEach(System.out::println);
}
```

> 该例子中，默认使用String实现的`Comparable`中的`compareTo`方法排序。

二、【`sorted(Comparator com)`】：

```java
userList.stream()
  .sorted((e1, e2) -> {
    if (e1.getAge().equals(e2.getAge())) {
      return e1.getName().compareTo(e2.getName());
    } else {
      return e1.getAge().compareTo(e2.getAge());
    }
  })
  .forEach(System.out::println);
```

### 2.3 步骤三：终止操作

#### 2.3.1 查找与匹配

`allMatch`：检查是否匹配所有元素。

`anyMatch`：检查是否至少匹配一个元素。

`noneMatch`：检查是否没有匹配所有元素。

`findFirst`：返回第一个元素。

`findAny`：返回当前流中任意元素。

`count`：返回流中元素的总个数。

`max`：返回流中的最大值。

`min`：返回流中的最小值。

示例：

模型准备：

```java
@Getter
@Setter
public class User {
    private String id;
    private String name;
    private Integer age;
    private Double salary;
    private Status status;

    public enum Status {
        BUSY,       // 繁忙
        FREE,       // 空闲
        VOCATION    // 假期
    }
}
```

数据源准备：

```java
List<User> userList = Arrays.asList(
  new User("101", "张三", 18, 9999.99, User.Status.FREE),
  new User("102", "李四", 59, 6666.66, User.Status.BUSY),
  new User("103", "王五", 28, 3333.33, User.Status.VOCATION),
  new User("104", "赵六", 8, 7777.77, User.Status.FREE),
  new User("105", "田七", 38, 5555.55, User.Status.BUSY)
);
```

一、【`boolean allMatch(Predicate<? super T> predicate);`】：是否匹配所有元素。

```java
boolean b1 = userList.stream()
                .allMatch((e) -> e.getStatus().equals(User.Status.BUSY));
```

> 判断是否匹配所有元素。结果为false。

二、【`boolean anyMatch(Predicate<? super T> predicate);`】：是否至少匹配一个元素。

```java
boolean b2 = userList.stream()
                .anyMatch((e) -> e.getStatus().equals(User.Status.BUSY));
```

> 判断是否至少匹配一个元素，返回true。

三、【`boolean noneMatch(Predicate<? super T> predicate);`】：是否没有匹配所有元素。

```java
boolean b3 = userList.stream()
                .noneMatch((e) -> e.getStatus().equals(User.Status.BUSY));
```

> 返回false。

四、【`Optional<T> findFirst();`】：返回第一个元素。

```java
Optional<User> first = userList.stream()
  .sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
  .findFirst();
```

> 取出工资最低的人。

五、【`Optional<T> findAny();`】：返回当前流中的任意元素。

```java
Optional<User> any = userList.stream()
  .filter((e) -> e.getStatus().equals(User.Status.FREE))
  .findAny();
```

> 从空闲状态的人中任意取出一个。

六、【`long count();`】：返回六中元素的总个数。

```java
long count = userList.stream().count();
```

七、【`Optional<T> max(Comparator<? super T> comparator);`】：返回流中最大值。

```java
Optional<User> max = userList.stream()
  .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
```

> 根据工资获取流中最大值。

八、【`Optional<T> min(Comparator<? super T> comparator);`】：返回流中最小值。

```java
Optional<Double> min = userList.stream()
                .map(User::getSalary) // 提取出工资
                .min(Double::compare);
```

> 获取流中最小的工资。

#### 2.3.2 归约与收集

归约：

- `reduce(T identity, BinaryOperator)`或`reduce(BinaryOperator)`：可以将**流中的元素反复结合起来，得到一个值。**

收集：

- `collect`：将流转换为其他形式，接受一个`Collector`接口的实现，用于给`Stream`中元素做汇总的方法。

一、`T reduce(T identity, BinaryOperator<T> accumulator);`：

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
Integer sum = list.stream()
                .reduce(0, (x, y) -> x + y);
```

> 将集合中的元素加起来。
>
> 第一个参数是起始值，将起始值0作为了x，从流中取出一个元素作为了y，进行x+y。将x+y的结果作为了x，从流中取出下一个元素，进行x+y。以此类推。

二、【`Optional<T> reduce(BinaryOperator<T> accumulator);`】：

```java
Optional<Double> reduce = userList.stream()
                .map(User::getSalary)
                .reduce((x, y) -> x + y);
```

> 为什么两个方法的返回值不一样呢？
>
> 因为第一个方法有起始值，其结果可能为空；而第二个方法结果有可能为空，所以用`Optional`封装，避免空指针。

备注：map和reduce的连接通常称为`map-reduce`模式，因Google用它来进行网络搜索而出名。

三、【`<R, A> R collect(Collector<? super T, A, R> collector);`】：

```java
// 把姓名提取出来转成集合
List<String> collect = userList.stream()
  .map(User::getName)
  .collect(Collectors.toList());

 // 总数
Long count = userList.stream()
  .collect(Collectors.counting());
// 平均值
Double avg = userList.stream()
  .collect(Collectors.averagingDouble(User::getSalary));
// 总和
Double sum = userList.stream()
  .collect(Collectors.summingDouble(User::getSalary));
// 最大值
Optional<User> max = userList.stream()
  .collect(Collectors.maxBy((user1, user2) -> user1.getSalary().compareTo(user2.getSalary())));
// 最小值
Optional<Double> min = userList.stream()
  .map(User::getSalary)
  .collect(Collectors.minBy(Double::compare));

// 根据状态分组
Map<User.Status, List<User>> map = userList.stream()
  .collect(Collectors.groupingBy(User::getStatus));
// 多级分组：先根据状态分，再根据年龄段分。
Map<User.Status, Map<String, List<User>>> map2 = userList.stream()
  .collect(Collectors.groupingBy(User::getStatus, Collectors.groupingBy((e) -> {
    if (e.getAge() <= 35) {
      return "青年";
    } else if (e.getAge() <= 50) {
      return "中年";
    } else {
      return "老年";
    }
  })));

// 分区：true一个区，false一个区
Map<Boolean, List<User>> map3 = userList.stream()
  .collect(Collectors.partitioningBy((e) -> e.getSalary() > 8000));

// 汇总
DoubleSummaryStatistics summaryStatistics = userList.stream()
  .collect(Collectors.summarizingDouble(User::getSalary));
System.out.println(summaryStatistics.getSum());
System.out.println(summaryStatistics.getAverage());
System.out.println(summaryStatistics.getCount());
System.out.println(summaryStatistics.getMax());
System.out.println(summaryStatistics.getMin());

// 拼接字符串
// 没有分隔符：张三李四王五赵六田七
String str1 = userList.stream()
  .map(User::getName)
  .collect(Collectors.joining());
// 使用","作为分隔符：张三,李四,王五,赵六,田七
String str2 = userList.stream()
  .map(User::getName)
  .collect(Collectors.joining(","));
// 使用","作为分隔符，首位添加"==="：===张三,李四,王五,赵六,田七===
String str3 = userList.stream()
  .map(User::getName)
  .collect(Collectors.joining(",", "===", "==="));
```

## 3 练习

一、【给定数字列表，如何返回一个由每个数的平方构成的列表。】

```java
Integer[] nums = new Integer[]{1, 2, 3, 4, 5};
Arrays.stream(nums)
  .map((x) -> x * x)
  .forEach(System.out::println);
```

二、【用map和reduce方法，数一数流中有多少个User。】：

```java
Optional<Integer> count = userList.stream()
  .map((e) -> 1)
  .reduce(Integer::sum);
System.out.println(count.get());
```

## 4 并行流和串行流

### 4.1 了解 Fork\Join框架

Fork\Join框架：就是在必要的情况下，将一个大任务，进行进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行join汇总。

Fork\Join框架和传统线程池的区别：采用”工作窃取“模式(`work-stealing`)：

当执行新的任务时，它可以将其拆分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。

> 相对于一般的线程池实现,fork/join框架的优势体现在对其中包含的任务的处理方式上.在一般的线程池中,如果一个线程正在执行的任务由于某些原因无法继续运行,那么该线程会处于等待状态.而在fork/join框架实现中,如果某个子问题由于等待另外一个子问题的完成而无法继续运行.那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程的等待时间,提高了性能。

### 4.2 并行流

并行流就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

Java 8 中将并行进行了优化，我们可以很容易的对数据进行并行操作。 Stream API 可以声明性地通过`parallel()`与`sequential()`在并行流与顺序流之间进行切换。 