---
title: 01 Java8新特性之Lambda 表达式
date: 2018-03-24
categories: Java新特性
tags: [Java8新特性, Lambda 表达式]
---

## 1 为什么使用Lambda表达式

Lambda 是一个**匿名函数**，我们可以把Lambda表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。可以写出更加简洁、灵活的代码。

<!-- more -->

## 2 Lambda 表达式引入案例

现有集合如下：

```java
List<User> userList = Arrays.asList(
  new User("101", "张三", 18, 9999.99),
  new User("102", "李四", 59, 6666.66),
  new User("103", "王五", 28, 3333.33),
  new User("104", "赵六", 8, 7777.77),
  new User("105", "田七", 38, 5555.55)
);
```

```java
public class User {
    private String id;
    private String name;
    private Integer age;
    private Double salary;
  	// 省略getter、setter和构造方法
}
```

有需求：获取公司中年龄小于35的员工信息。

一、【原始写法】：

```java
public List<User> filterUsersAge(List<User> users) {
  List<User> list = new ArrayList<>();
  for (User user : users) {
    if (user.getAge() <= 35) {
      list.add(user);
    }
  }
  return list;
}
```

测试：

```java
@Test
public void test1() {
  List<User> list = filterUsersAge(userList);
  for (User user : list) {
    System.out.println(user);
  }
}
```

但是此时如果需求变了，要求获取公司中工资大于 5000 的员工信息。这是就需要重写一个方法来完成需求。这样做自然不好。

二、【优化方式一】：策略设计模式

> 策略模式，就是将一个算法的不同实现封装成一个个单独的类，这些类实现同一个接口，使用者直接使用该接口来访问具体的算法。这个样子，使用者就可以使用不同的算法来实现业务逻辑了。

首先定义一个接口：

```java
public interface MyPredicate<T> {
  public boolean compare(T t);
}
```

然后根据需求`获取公司中年龄小于35的员工信息`，编写实现类：

```java
public class FilterUsersForAge implements MyPredicate<User> {
  @Override
  public boolean compare(User user) {
    return user.getAge() <= 35;
  }
}
```

优化原始的写法：

```java
public List<User> filterUsers(List<User> users, MyPredicate<User> myPre) {
  List<User> list = new ArrayList<>();
  for (User user : users) {
    if (myPre.compare(user)) {
      list.add(user);
    }
  }
  return list;
}
```

测试：

```java
@Test
public void test2() {
  List<User> users = filterUsers(userList, new FilterUsersForAge());
  for (User user : users) {
    System.out.println(user);
  }
}
```

此时如果需要改成了`获取公司中工资大于 5000 的员工信息`。只需要重写一个类实现`MyPredicate`。

```java
public class FilterUsersForSalary implements MyPredicate<User> {
  @Override
  public boolean compare(User user) {
    return user.getSalary() >= 5000;
  }
}
```

测试：

```java
@Test
public void test2() {
  List<User> users = filterUsers(userList, new FilterUsersForSalary());
  for (User user : users) {
    System.out.println(user);
  }
}
```

> 调用`filterUsers`方法时使用不同的实现类做形参即可。

三、【优化方式二】：匿名内部类

虽然`优化方式一`的策略设计模式很好的解决了需求。但是如果两个实现类都只调用一次的话，这样写就太浪费了。如果改类只用一次的话，则可以使用匿名内部类代替。

```java
@Test
public void test3() {
  List<User> list = filterUsers(userList, new MyPredicate<User>() {
    @Override
    public boolean compare(User user) {
      return user.getAge() <= 35;
    }
  });

  for (User user : list) {
    System.out.println(user);
  }
}

@Test
public void test4() {
  List<User> list = filterUsers(userList, new MyPredicate<User>() {
    @Override
    public boolean compare(User user) {
      return user.getSalary() >= 5000;
    }
  });

  for (User user : list) {
    System.out.println(user);
  }
}
```

**四、【优化方式三】：Lambda 表达式**

虽然`优化方式二`比`优化方式一`的代码简洁了不少，但是我们会发现其中有不少都是重复的代码。通过观察发现，其实有用的代码只有`user.getAge() <= 35`和`user.getSalary() >= 5000`两句。

因此可以使用Lambda 表达式。

```java
@Test
public void test5() {
  List<User> users = filterUsers(userList, (e) -> e.getAge() <= 35);
  users.forEach(System.out::println);

  System.out.println("----------------");

  List<User> users2 = filterUsers(userList, (e) -> e.getSalary() >= 5000);
  users2.forEach(System.out::println);
}
```

五、【优化方式四】：Steam API（下一篇介绍）

```java
@Test
public void test6() {
  userList.stream().filter((e) -> e.getAge() <= 50)
          			.forEach(System.out::println);
}
```

## 3 Lambda 表达式基础语法

**Java8中引入了新的操作符 `->`** ，称作箭头操作符或 Lambda 操作符。

箭头操作符将 Lambda 表达式拆分成两部分：

- 左侧：Lambda 表达式的参数列表。

  > 参数列表对应的是接口中抽象方法的参数列表。

- 右侧：Lambda 表达式中所需要执行的功能。即 Lambda 体。

  > 对接口中抽象方法的实现功能。

### 3.1 语法格式一：无参数、无返回值

> 接口中的抽象方法**无参数、无返回值**。

`() -> System.out.println("hello Lambda");`

示例：

```java
@Test
public void test1() {
  // 传统写法
  Runnable r = new Runnable() {
    int num = 0;
    
    @Override
    public void run() {
      System.out.println("hello Lambda!" + num);
    }
  };
  r.run();
  System.out.println("=====================");
  
  // Lambda 表达式写法
  Runnable r1 = () -> System.out.println("hello Lambda!" + num);
  r1.run();
}
```

> 注意事项：如果在匿名内部类中使用了同级别的局部变量时，在JDK1.8以前该变量必须加上final修饰的。**但从jdk1.8开始，该变量可以省略final修饰，其已经默认为final变量，不能修改。**

### 3.2 语法格式二：有参数、无返回值

> 接口中的抽象方法**有一个参数，无返回值。**

```java
@Test
public void test2() {
  Consumer<String> con = (x) -> System.out.println(x);
  con.accept("hello Lambda");
}
```

> 执行程序，控制台打印：hello Lambda

### 3.3 语法格式三：若只有一个参数，小括号可以省略

> 接口中的抽象方法**若只有一个参数，小括号可以省略不写。**

语法格式二可以写成：`Consumer<String> con = x -> System.out.println(x);`

### 3.4 语法格式四：有两个以上参数，Lambda 体中有多条语句，有返回值

> 接口中的抽象方法**有两个以上参数，Lambda 体中有多条语句，有返回值。**

```java
@Test
public void test3() {
  Comparator<Integer> con = (x, y) -> {
    System.out.println("hello Lambda");
    return Integer.compare(x, y);
  };
}
```

> 注意：**如果有多条语句，Lambda 体必须使用大括号`{}`。**

### 3.5 语法格式五：只有一条语句，retrun和大括号可以省略

> **若Lambda 体中只有一条语句，return和`{}`都可以不省略不写。**

```java
@Test
public void test3() {
  Comparator<Integer> con = (x, y) -> Integer.compare(x, y);
}
```

### 3.6 语法格式六：类型推断

Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器可以通过上下文推断出数据类型，即`类型推断`。

### 3.7 语法格式七：需要函数式接口的支持

**函数式接口：若接口中只有一个抽象方法的接口，成为函数式接口。**可以使用注解`@FunctionalInterface`修饰，可以检查是否是函数式接口。

> 换句话说，使用`@FunctionalInterface`修饰的接口必须是函数式接口，接口中只能有一个抽象方法。

## 4 四大内置核心函数式接口

一、【`Consumer<T>`】：消费型接口。

抽象方法：`void accept(T t);`

二、【`Supplier<T>`】：供给型接口。

抽象方法：`T get();`

三、【`Function<T,R>`】：函数型接口。

抽象方法：`R apply(T t);`

四、【`Predicate<T>`】：断言型接口。

抽象方法：`boolean test(T t);`

除了以上接口，还有以上接口的子接口，用法差不多。

## 5 方法引用和构造器引用

### 5.1 方法引用（method reference）

若Lambda 体中的内容有方法已经实现了，我们可以使用`方法引用`。

> 可以理解为方法引用是 **Lambda 表达式的另外一种表现形式。**

主要有三种语法格式：

1. `对象::实例方法名`
2. `类::静态方法名`
3. `类::实例方法名`

方法引用的**前提条件**：

1. **实现抽象方法中的参数列表和返回值类型得与 Lambda 体中已实现方法的参数列表和返回值类型一致。**
2. 若 Lambda 参数列表中的第一个参数是实例方法的调用者，二第二个参数是实例方法的参数时，可以使用`类::实例方法名`。

一、【`对象::实例方法名`】：

```java
@Test
public void test1() {
  // Lambda 表达式
  Consumer<String> con = (x) -> System.out.println(x);
  // 方法引用
  Consumer<String> con1 = System.out::println;
}

@Test
public void test2() {
  User user = new User();
  // Lambda 表达式
  Supplier<String> sup = () -> user.getName();
  // 方法引用
  Supplier<String> sup2 = user::getName;
}
```

二、【`类::静态方法名`】：

```java
@Test
public void test3() {
  Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
  // 方法引用
  Comparator<Integer> com2 = Integer::compare;
}
```

三、【`类::实例方法名`】：

```java
@Test
public void test4() {
  BiPredicate<String, String> bp = (x, y) -> x.equals(y);
  // 方法引用
  BiPredicate<String, String> bp2 = String::equals;
}
```

> 注意： Lambda 表达式的第一个参数是方法的调用者，第二个参数是方法的参数时，就可以使用`类::实例方法名`的方式。
>
> 其中，x是`equals`方法的调用者，y是`equals`方法的参数。

### 5.2 构造器引用

格式：`类::new`。

注意：需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致。

```java
@Test
public void test5() {
  Supplier<User> sup = () -> new User();
  // 构造器引用
  Supplier<User> sup2 = User::new; // 无参构造
  User user = sup2.get();
}
```

```java
// 若User有构造器
public User(String id) {
  this.id = id;
}
public User(String id, Integer age) {
  this.id = id;
  this.age = age;
}

@Test
public void test6() {
  Function<String, User> fun = (x) -> new User(x);
  // 构造器引用
  Function<String, User> fun2 = User::new; // 一个参数构造
  fun2.apply("101");
  
  BiFunction<String, Integer, User> bf = (x, y) -> new User(x, y);
  BiFunction<String, Integer, User> bf2 = User::new; // 两个参数构造
}
```

### 5.3 数组引用

语法：`类型[]::new`

```java
@Test
public void test7() {
  Function<Integer, String[]> fun = (x) -> new String[x];
  // 数组引用
  Function<Integer, String[]> fun2 = String[]::new;
  String[] arr = fun2.apply(20); // 数组的长度为20
}
```

