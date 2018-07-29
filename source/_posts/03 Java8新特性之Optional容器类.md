---
title: 03 Java8新特性之Optional容器类
date: 2018-04-25
categories: Java新特性
tags: [Java8新特性, Lambda 表达式, Optional]
---

`Optional<T>`类是一个容器类，代表一个值存在或者不存在。原来用null表示一个值不存在，现在`Optional`可以更好的表达这个概念。并且可以避免空指针异常。

> 容器类是用来封装对象的。泛型T就是需要封装的类。

<!-- more -->

## 1 Optional类常用的方法

一、`Optional.of(T t)`：创建一个`Optional`实例。

```java
Optional<User> userOptional = Optional.of(new User());
User user = userOptional.get();
System.out.println(user);
```

> 注意：在实际开发中，`of`方法的形参从其他地方传入时有可能为null，此时会抛出空指针异常。

二、`Optional.empty()`：创建一个空的`Optional`实例。

```java
Optional<User> emptyOptional = Optional.empty();
User user = emptyOptional.get();
```

> 在第二局抛出异常：`java.util.NoSuchElementException: No value present`。

三、`Optional.ofNullable(T t)`：若t不为null，创建Optional实例。否则创建空实例。

```java
Optional<Object> nullableOptional = Optional.ofNullable(null);
Object o = nullableOptional.get();
```

> 在第二局抛出异常：`java.util.NoSuchElementException: No value present`。

`ofNullable()`方法结合了`of()`和`empty()`方法，源码如下：

```java
public static <T> Optional<T> ofNullable(T value) {
  return value == null ? empty() : of(value);
}
```

四、`boolean isPresent()`：判断是否包含值。

```java
Optional<Object> nullableOptional = Optional.ofNullable(null);
if (nullableOptional.isPresent()) { // 不包含值，为false
  Object o = nullableOptional.get();
}
```

五、`T orElse(T other)`：如果调用对象包含值，返回该值，否则返回other。

```java
Optional<User> nullableOptional = Optional.ofNullable(new User());
User user = nullableOptional.orElse(new User("123", "嘻嘻", 21, 1000.0));
```

> 如果有值，返回该值，否则用`new User("123", "嘻嘻", 21, 1000.0)`代替。

六、`T orElseGet(Supplier<? extends T> other)`：如果调用对象包含值，返回该值，否则返回other获取的值。

```java
Optional<User> nullableOptional = Optional.ofNullable(new User());
User user1 = nullableOptional.orElseGet(() -> new User());
```

七：`<U> Optional<U> map(Function<? super T, ? extends U> mapper)`：如果有值对其处理，并返回处理后的`Optional`，否则返回`Optional.empty()`。

```java
Optional<User> userOptional = Optional.of(new User("123", "嘻嘻", 21, 1000.0));
Optional<String> nameOptional = userOptional.map((user) -> user.getName());
System.out.println(nameOptional.get());
```

八、`<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)`：与map类似。要求Function函数式接口的返回值必须是`Optional`。

```java
Optional<User> userOptional = Optional.of(new User("123", "嘻嘻", 21, 1000.0));
Optional<String> nameOptional = userOptional.flatMap((user) -> Optional.of(user.getName()));
System.out.println(nameOptional.get());
```

## 2 Optional的最佳实践

### 2.1 Optional应该只用于返回类型

`Optional`值应该在遇到它们的地方中处理。

### 2.2 不要简单的调用`get()`方法

`Optional`的目的是用来表示此值很可能为空。所以，在使用它之前进行检查是很有必要的。在某些情况下简单的调用`get()`方法但没有使用`isPresent()`方法进行检查时，同样会导致空指针异常。

或者你不使用`isPresent()`方法来进行检查，可以使用`orElse()`或者`orElseGet()`方法来给出一个代替的值，来解决空指针的问题。