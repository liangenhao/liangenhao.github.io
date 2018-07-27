---
title: 03 Spring 注解驱动开发之属性赋值
date: 2018年5月31日
categories: Spring Boot
tags: [Spring Boot, Annotation]
---

## 1 @Value 赋值和 @PropertySource 加载属性文件

`@Value`注解的唯一属性`value`用来赋值。

` @PropertySource`注解用来加载属性文件。使用`${}`可以取出配置文件里的值。

<!-- more -->

一、【基本数值】：

```java
@Value("张三")
private String name;
```

二、【SpELl 表达式赋值：`#{}`】：

```java
@Value("#{100-2}")
private Integer age;
```

三、【取配置文件(properties)中的值进行赋值：`${}`】：

1. 首先创建`person.properties`配置文件：

   ```properties
   person.nickName=咿呀咿呀呦
   ```

2. 在spring配置文件中加载配置文件：

   ```xml
   <context:property-placeholder location="classpath:person.properties"/>
   ```

   或者，在配置类中使用`@PropertySource`加载配置文件：

   ```java
   @Configuration
   @PropertySource(value = {"classpath:person.properties"})
   public class MainConfigOfPropertyValues {
     @Bean
     public Person person() {
       return new Person();
     }
   }
   ```

   > `@PropertySource`的`value`属性的值是数组，这就意味着可以加载多个配置文件。又因为该注解是重复注解，所以也可以多次使用该注解加载不同的配置文件。还可以使用`@PropertySources`注解，该注解的`value`属性可以加载多个`@PropertySource`注解。
   >
   > 类路径下的文件使用`classpath:`标注。文件路径下的文件使用`file:`标注。

3. 使用`@Value`赋值：

   ```java
   @Value("${person.nickName}")
   private String nickName;
   ```

【注】：Spring 中 `#{}`和`${}`的区别：

1. `${key}`通常用来获取属性文件中的内容。
2. `#{表达式}`是SpEL表达式的格式。
3. 二者可以混合使用：`#{'${key}'}`

