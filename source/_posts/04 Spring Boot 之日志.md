---
title: 04 Spring Boot 1.x 之日志
date: 2018-09-01
categories: Spring Boot
tags: [Spring Boot]
---



[TOC]

## 1 日志框架

| 日志门面 （日志抽象层）                                      | 日志实现                                                 |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| `JCL(Jakarta Commons Logging)`、` SLF4j(Simple Logging Facade for Java)`、` jboss-logging` | `Log4j `、`JUL(java.util.logging)`、`Log4j2`、 `Logback` |

左边选一个抽象层，右边选一个实现：

最常用的日志门面：`SLF4J`。

日志实现：`Log4J`和`Logback`是一个写的框架。`Log4J2`是Apache出的。

**Spring Boot 选用的是`SLF4J`和`Logback`。**

## 2 SLF4J 使用

![slf4J](04 Spring Boot 之日志\slf4J.png)

日志记录方法的调用，不应该直接调用日志的实现类，而是调用日志抽象层的方法（和JDBC一样，面向接口编程）。

需要给项目里导入`slf4J`和`logback`的jar。

``` java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(HelloWorld.class);
        logger.info("Hello World");
    }
}
```

每个日志框架都有自己的配置文件，使用`slf4j`以后，配置文件还是做出日志实现框架自己本身的配置文件。

## 3 遗留问题：统一日志

在项目中我们会用到不同的框架，但这些不同的框架各自所使用的日志框架也是不同的，例如 Spring 用的是`commons-logging`，hibernate 用的是 `jboss-logging`。

所以需要统一日志记录，统一使用`slf4j`和`logback`。

下面这张图展示了怎样统一使用`slf4j`日志。

![legacy](04 Spring Boot 之日志\legacy.png)

步骤一：将系统中其他日志框架先排除除去。

步骤二：用中间包来替换原有的日志框架。

步骤三：导入`slf4j`包和其他的实现日志包。

## 4 Spring Boot 日志关系

Spring Boot 使用`spring-boot-starter-logging`来做日志功能。

Spring Boot 日志的依赖关系：

![spring-boot-starter-logging](04 Spring Boot 之日志\spring-boot-starter-logging.png)

一、Spring Boot 底层使用的是`slf4j`+`logback`的方式进行日志记录。

二、Spring Boot 也把其他的日志都替换成了`slf4j`。

**三、如果我们要引入其他框架，一定要把这个框架的默认日志依赖移除掉。**

## 5 Spring Boot 日志默认配置

Spring Boot 默认已经配置好了日志了，默认的日志级别是`info`。

> 日志级别由低到高：trace < debug < info < warn < error。

### 5.1 修改日志级别

格式：`logging.level.* = LEVEL `

> `logging.level`：日志级别控制前缀，*为包名或Logger名 。
>
> `LEVEL`：选项`TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF` 

`logging.level.root=debug`：配置root的日志级别。

`logging.level.包名=debug`：配置指定包下的日志级别。

### 5.2 指定日志输出位置

如果不指定`logging.file`或者`logging.path`，日志只在控制台输出。

使用`logging.file`指定文件名，将日志输出到指定的文件中。可以是绝对路径，也可以是相对路径。

> `logging.file=my.log`：日志放在当前项目根路径下。
>
> `logging.file=/my.log`：日志放在当前磁盘根路径下。

使用`logging.path`指定目录。

> `logging.path=/spring/log`：在当前磁盘的根路径下创建spring文件夹和log文件夹，日志文件使用默认的`spring.log`。

如果`logging.file`和`logging.path`同时配置，只有`logging.file`起作用。

### 5.3 指定日志输出的格式

`logging.pattern.console`：指定在控制台输出的日志的格式。

`logging.pattern.file`：指定文件中日志输出的格式。

## 6 指定日志框架配置文件

给类路径下放上日志框架自己的配置文件即可，Spring Boot 就不使用他默认的配置了。

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

**推荐使用带有`-spring`后缀的日志名称。**

如果使用`logback.xml`名称的，就直接被日志框架识别了。

如果使用`logback-spring.xml`带有`-spring`后缀的日志名称，日志框架就不直接加载日志的配置文件了，由Spring Boot 加载，**因此可以使用Spring Boot的多环境配置功能`springProfile`。**

```xml
<springProfile name="dev">
	...
</springProfile>
```

可以指定某段配置只在某个环境下生效。通过`spring.profiles.active`来配置当前激活的profile。

## 7 切换日志框架

### 7.1 切换到 log4j (不推荐)

按照`slf4j`的日志是配图，进行相关切换。

`slf4j` + `log4j`的方式：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>logback-classic</artifactId>
            <groupId>ch.qos.logback</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j-over-slf4j</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

切换到`log4j`，需要创建`log4j`的日志配置文件`log4j.properties`。

### 7.2 切换到 log4j2

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

