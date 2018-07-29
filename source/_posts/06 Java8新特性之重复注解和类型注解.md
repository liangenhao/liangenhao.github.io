---
title: 06 Java8新特性之重复注解和类型注解
date: 2018-05-13
categories: Java新特性
tags: [Java8新特性, Annotation]
---

## 1 重复注解

重复注解：允许在同一声明类型（类，属性，或方法）上多次使用同一个注解。**重复注解本身必须使用`@Repeatable`注解。**

<!-- more -->

一、步骤一：自定义注解`MyAnnotation`和包装类注解`MyAnnotations`，包装类注解用来放置一组具体的`MyAnnotation`注解：

```java
@Repeatable(MyAnnotations.class)
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
  String value() default "enhao";
}

@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotations {
  MyAnnotation[] value();
}
```

二、使用注解：

```java
public class TestAnnotation {
    @MyAnnotation("hello")
    @MyAnnotation("World")
    public void show() {
    }
}
```

测试：

```java
@Test
public void test1() throws NoSuchMethodException {
  Class<TestAnnotation> clazz = TestAnnotation.class;
  Method method = clazz.getMethod("show");
  MyAnnotation[] annotations = method.getAnnotationsByType(MyAnnotation.class);
  for (MyAnnotation myAnnotation : annotations) {
    System.out.println(myAnnotation.value());
  }
}
```

> 可以获取`show`方法上注解的两个值。

## 2 类型注解

Java 8的类型注解扩展了注解使用的范围。新增`ElementType.TYPE_USE`和`ElementType.TYPE_PARAMETER`。（在`@Target`注解上）

`ElementType.TYPE_USE`表示该注解能写在使用类型的任何语言中，如声明语句、泛型和强制转换语句中的类型。

`ElementType.TYPE_PARAMETER`表示该注解能写在类型变量的声明语句中。

```java
//初始化对象时  
String myString = new @NotNull String();  
//对象类型转化时  
myString = (@NonNull String) str;  
```

### 2.1 类型注解的作用

引用 [扩展注解](https://blog.csdn.net/sun_promise/article/details/51315032) 中的内容：

类型注解被用来支持在java中做强类型检查。配合第三方工具 Checker Framework。可以在编译的时候检测出runtime error（`UnsupportedOperationException`； `NumberFormatException`；`NullPointerException`异常等都是runtime error)。

> 注意：java 5,6,7版本是不支持注解`@NonNull`，但checker framework 有个向下兼容的解决方案，就是将类型注解@NonNull 用/**/注释起来。
>
> 这样javac编译器就会忽略掉注释块，但用checker framework里面的javac编译器同样能够检测出@NonNull错误。
> 通过 **类型注解 + checker framework **可以在编译时就找到runtime error。