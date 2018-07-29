---
title: 04 Java8新特性之接口中的默认方法与静态方法
date: 2018-05-05
categories: Java新特性
tags: [Java8新特性, interface]
---

在Java8之前，接口中可以包含的成员有：抽象方法和常量。抽象方法的默认修饰是`public abstract`。常量的默认修饰是`public static final`。并且这些默认修饰符是可以省略不写的。

在Java8中接口可以包含具有具体实现的方法。有默认方法和静态方法两种。

<!-- more -->

## 1 接口中的默认方法

默认方法，使用`default`修饰符。`default`不可省略。

### 1.1 “类优先”原则

若一个接口中定义了默认方法。而另外一个父类或者接口中又定义了一个同名的方法时：

- 选择父类中的方法。如果一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略。 
- 接口冲突。如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法（不管方法是否是默认方法）， 那么必须覆盖该方法来解决冲突 。

一、接口中定义了默认方法，父类中也又相同名称和参数的方法：

```java
// 接口中定义了一个默认方法
public interface MyInterface {
  default String getName() {
    return "this is my interface!";
  }
}

// 父类中有一个和接口中的默认方法同名同参数的方法
public class SuperClass {
  public String getName() {
    return "this is super class!";
  }
}

// 子类同时继承父类和实现接口，使用的是父类中的方法，接口中的默认方法会被忽略
public class SubClass extends SuperClass implements MyInterface {    
}
```

二、一个接口中定义了一个默认方法，另一个接口中也定义了一个同名同参数的方法

```java
// 接口中定义了一个默认方法
public interface MyInterface {
  default String getName() {
    return "this is my interface!";
  }
}

// 另一个接口中定义了一个同名同参数的默认方法
public interface MyOtherInterface {
  default String getName() {
    return "this is my other interface!";
  }
}
```

此时，有一个子类同时实现了这两个接口：

```java
public class AnotherSubClass implements MyInterface, MyOtherInterface {
}
```

此时编译器会报错：

![java8interface](04 Java8新特性之接口中的默认方法与静态方法\java8interface.png)

此时必须重写该方法来解决冲突问题。

若想选择某一个接口中的默认方法，重写的方法可以为：

```java
public class AnotherSubClass implements MyInterface, MyOtherInterface {
  @Override
  public String getName() {
    return MyInterface.super.getName(); // 使用的是MyInterface中默认方法
  }
}
```

## 2 接口中的静态方法

Java8中接口可以定义静态方法，使用`static`修饰。

```java
public interface MyInterface {
  static void show() {
    System.out.println("this is myInterface static method");
  }
}
```

