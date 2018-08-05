---
title: 01 Spring Boot 之HelloWorld
date: 2018-08-05
categories: Spring Boot
tags: [Spring Boot]
---

## 1 Spring Boot 简介

Spring Boot是用来简化Spring 应用的开发，**约定大于配置**的原则，去繁为简。Spring Boot 通过整合Spring 的整个技术栈，来完成和简化企业级的开发。

Spring Boot使用嵌入式的Servlet容器，应用无需打成war包。

Spring Boot的启动器starters，可以进行自动的依赖管理和版本控制。

Spring Boot包含了大量的自动配置。也可以修改默认值。无需配置xml，无代码生成。

Spring Boot有准生产环境的运行时应用监控。

<!-- more -->

##  2 微服务 Microservices

简而言之，微服务架构风格是一种将单个应用程序作为一套小型服务开发的方法，每种小型服务都运行在自己的进程中，并与轻量级机制（通常是HTTP资源API）进行通信。 这些服务是围绕业务功能构建的，可以通过全自动部署机制独立部署。 这些服务的集中管理最少，可以用不同的编程语言编写，并使用不同的数据存储技术。

Martin Fowler关于微服务的原文：

```
In short, the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.
```

有篇翻译自Martin Fowler关于微服务文章的翻译，有时间可以读下：[微服务](http://blog.cuicc.com/blog/2015/07/22/microservices/)。

## 3 开发环境

JDK：1.8

maven：3.3.9

IntelliJ IDEA 2017

Spring Boot 2.x RELEASE

> 需要注意的是：Spring Boot 2 底层是 Spring framework 5， 必须使用jdk8以上版本，maven 3.2以上版本。

## 4 Spring Boot Hello World

### 4.1 创建 Spring Boot 项目

使用Spring官方的[Spring Initializr](https://start.spring.io/)。快速搭建 Spring Boot 工程。

`resources`文件夹的目录结构：

- `static`：保存所有的静态资源：js、css、images。

- `templates`：保存所有的模板页面。（Spring Boot 默认是jar包使用嵌入式的Tomcat，默认不支持JSP页面，可以使用模板引擎（thymeleaf、freemarker））

  > `templates`目录下的文件不能直接访问，需要通过Controller做跳转。

- `application.properties`：Spring Boot 应用的配置文件。

### 4.2 细节探究

#### 4.2.1 POM 文件

一、【父项目】

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

Spring Boot的工程需要依赖一个父项目。我们查看这个父项目的pom文件，发现它也依赖了一个父工程：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

查看这个`spring-boot-dependencies`项目的pom文件，发现该pom文件里定义了Spring Boot整合的所有依赖的版本号，这就意味着，**我们在自己的Spring Boot项目里引用Spring Boot整合的依赖时，不用再指定版本号。**

> 因此，`spring-boot-dependencies`又被称为Spring Boot 的版本仲裁中心。

二、【导入的依赖】：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

`spring-boot-starter`：Spring Boot 场景启动器。

`spring-boot-starter-web`：帮我们导入了web模块正常运行所依赖的组件。

Spring Boot 将所有的功能场景都抽取出来，做成一个个starters，只需要在项目里引入这些 starter ，相关场景的所有依赖都会导入进来。版本由Spring Boot自动控制。

#### 4.2.2 主程序类：主入口类

```java
@SpringBootApplication
public class SpringBoot01HelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01HelloworldApplication.class, args);
    }
}
```

`@SpringBootApplication`：该注解标注在某个类上，说明这个类是 Spring Boot 的主配置类。Spring Boot 就应该运行这个类的main方法来启动 Spring Boot 应用。

我们查看`@SpringBootApplication`注解的源码，发现它是一个组合注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

`@SpringBootConfiguration`：Spring Boot 配置类注解，标注在某个类上，表示这是一个Spring Boot 的配置类。其实查看该注解的源码，发现该注解被标注了`@Configuration`注解，这是 Spring 配置类的注解。

**`@EnableAutoConfiguration`：开启自动配置功能**。查看该注解的源码：

> ```java
> @AutoConfigurationPackage
> @Import(EnableAutoConfigurationImportSelector.class)
> public @interface EnableAutoConfiguration {}
> ```
>
> `@AutoConfigurationPackage`：自动配置包。作用：**将主配置类(`@SpringBootApplication`标注的类)的所在包及下面所有子包里面的所有组件扫描到Spring容器中。**
>
> `@Import(EnableAutoConfigurationImportSelector.class)`：导入自动配置类的选择器。会给容器**导入自动配置类**`XxxAutoConfiguration`。Spring Boot 在启动的时候从类路径下的`META-INF/spring.factories`中获取`EnableAutoConfiguration`指定的值，将这些值作为自动配置类导入到容器中，自动配置类生效，帮我们进行自动配置工作。 自动配置都在`spring-boot-autoconfigure-2.x.x.RELEASE.jar`包下。

