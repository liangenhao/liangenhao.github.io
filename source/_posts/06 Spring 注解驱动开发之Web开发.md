---
title: 06 Spring 注解驱动开发之Web开发
date: 2018-06-16
categories: Spring Boot
tags: [Spring Boot, Annotation]
---

在 Servlet3.0 之前，使用web的三大组件：servlet、filter、listener都需要在web.xml中进行注册配置。

在 Servlet3.0 标准发布之后，提供了注解的支持，异步处理的支持和可插拔的插件的支持。

> Tomcat7.0以上的版本才支持Servlet3.0标准。

<!-- more -->

## 1 Servlet3.0中ServletContainerInitializer

> Servlet3.0标准中引入了一个新的内容：`shared libraries`和`runtimes pluggability`。

Servlet 容器启动时会扫描当前应用里面每一个jar包的`ServletContainerInitializer`的实现类。 并且`ServletContainerInitializer`的**实现类，必须绑定在`META-INF/services/javax.servlet.ServletContainerInitializer`。文件的内容就是`ServletContainerInitializer`实现类的全类名。**

简单说：容器在启动应用的时候，会扫描当前应用每一个jar包里面`META-INF/services/javax.servlet.ServletContainerInitializer`文件中指定的实现类，**启动并运行这个实现类中的方法**`onStartup`。

`ServletContainerInitializer`的实现类是可以使用注解`@HandlersTypes`注解，作用是指定需要处理类型，容器启动的时候 将`@HandlersTypes`注解中指定的类型下的子类（实现类和子接口等）传递过来。

`onStartup`方法有两个参数：

- `Set<Class<>> set`：需要处理的类型。是`@HandlersTypes`注解指定的类型的子类或者子接口。 
- `ServletContext servletContext`：代表当前web应用的`ServletContext`对象。（一个Web应用对应一个`ServletContext`对象）。可以使用它来注册三大组件。

```java
@HandlesTypes(value = {PersonService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {

  /**
    *
    * @param set 需要处理的类型。是@HandlersTypes注解指定的类型的子类或者子接口。 
    * @param servletContext 代表当前web应用的ServletContext对象。（一个Web应用对应一个ServletContext对象）
    * @throws ServletException
    */
  @Override
  public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {

  }
}
```



## 2 整合Spring MVC

以前用xml的配置方式，需要在`web.xml`中先配置`ContextLoaderListener`监听器，来加载Spring的父容器。然后配置`DispatcherServlet`来配置Spring MVC前端控制器，加载子容器。

### 2.1 整合分析

![javax.servlet.ServletContainerInitializer](06 Spring 注解驱动开发之Web开发\javax.servlet.ServletContainerInitializer.png)

我们可以查看`spring-web-4.x.x.RELEASE`包中`META-INF/services/javax.servlet.ServletContainerInitializer`文件里的内容：

```
org.springframework.web.SpringServletContainerInitializer
```

这就意味着在web容器启动的时候，会加载`SpringServletContainerInitializer`类：

该实现类上标注了`@HandlesTypes({WebApplicationInitializer.class})`。则Spring应用一启动，会加载`WebApplicationInitializer`接口下的子类和子接口。并且为不是接口和抽象类的`WebApplicationInitializer`组件创建对象。![WebApplicationInitializer创建对象](06 Spring 注解驱动开发之Web开发\WebApplicationInitializer创建对象.png)

`WebApplicationInitializer`接口有三个抽象实现：

`AbstractContextLoaderInitializer`：第一层抽象类，创建`RootApplicationContext`根容器。

`AbstractDispatcherServletInitializer`：第二层抽象类，创建一个web的容器。并创建一个`DispatcherServlet`，将创建的`DispatcherServlet`添加到`ServletContext`中。

`AbstractAnnotationConfigDispatcherServletInitializer`：第三层抽象类。注解方式创建根容器、创建`DispatcherServlet`。

【总结】：**以注解方式来启动springmvc，我们自己的配置类继承`AbstractAnnotationConfigDispatcherServletInitializer`，并实现抽象方法，指定`DispatcherServlet`的配置信息。**

附 spring官方文档中推荐的配置方式：

![spring官方配置方式](06 Spring 注解驱动开发之Web开发\spring官方配置方式.png)

> 以父子容器的形式配置，web容器用来扫描Controller，视图解析器，映射等等web相关功能组件。根容器用来扫描业务逻辑组件、数据源、事务等等。

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { RootConfig.class };
  }

  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class<?>[] { App1Config.class };
  }

  @Override
  protected String[] getServletMappings() {
    return new String[] { "/app1/*" };
  }
}
```

### 2.2  整合

一、根容器配置类：

```java
// Spring的根容器不扫描标注了@Controller注解的类。
@ComponentScan(value = "com.enhao.spring.mvc.annotation", excludeFilters = {
  @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
public class RootConfig {
}
```



二、子容器配置类：

```java
// spring mvc子容器只扫描标注了@Controller注解的类，警用默认的过滤规则。
@ComponentScan(value = "com.enhao.spring.mvc.annotation", includeFilters = {
  @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
},useDefaultFilters = false)
public class WebConfig {
}
```



三、创建自定义初始化类，继承`AbstractAnnotationConfigDispatcherServletInitializer`抽象类：

```java
// web容器启动的时候创建对象，调用方法来初始化容器以及前端控制器
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
  /**
     * 获取根容器的配置类：类似于读取Spring的配置文件（创建出父容器）
     * @return
     */
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class[]{RootConfig.class};
  }

  /**
     * 获取web容器的配置类：类似于读取Spring MVC配置文件（创建出子容器）
     * @return
     */
  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class[]{WebConfig.class};
  }

  /**
     * 获取DispatcherServlet的映射信息
     * @return
     */
  @Override
  protected String[] getServletMappings() {
    // "/"：拦截所有请求，包含静态资源（xx.js, xx.png），但不包括*.jsp文件。
    // "/*"：拦截所有请求，包含*.jsp
    return new String[]{"/"};
  }
}
```

### 2.3 定制SpringMVC的配置

一、使用**`@EnableWebMvc`注解**，开启SpringMVC定制配置。相当于`<mvc:annotation-driven />`。

二、配置组件：视图解析器、视图映射、静态资源映射、拦截器等等。

配置类实现`WebMvcConfigurer`接口，就可以配置以上提到的内容。

但实现`WebMvcConfigurer`接口里的方法太多了，所以我们**通常继承`WebMvcConfigurerAdapter`抽象类**，它是`WebMvcConfigurer`的一个空实现。

示例：

```java
@ComponentScan(value = "com.enhao.spring.mvc.annotation", excludeFilters = {
  @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
@EnableWebMvc
public class RootConfig extends WebMvcConfigurerAdapter {

  // 配置视图解析器
  @Override
  public void configureViewResolvers(ViewResolverRegistry registry) {
    // 默认页面都从/WEB-INF/xxx.jsp
    // registry.jsp();
    registry.jsp("/WEB-INF/views", ".jsp");
  }

  // 静态资源访问：相当于：<mvc:default-servlet-handler/>
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }

}
```

## 3 异步请求

Spring MVC的异步处理是基于Servlet3.0的异步处理。

一、方式一：控制器返回`Callable`：

1. 控制器返回`Callable`。
2. SpringMVC就会异步的将`Callable`提交到`TaskExecutor`使用一个隔离的线程进行执行。
3. `DispatcherServlet`和所有的`Filter`退出web容器的线程，但response保持打开状态。
4. `Callable`返回结果，SpringMVC将请求重新派发给容器，恢复之前的请求。
5. 根据`DispatcherServlet`返回的结果，SpringMVC继续进行视图渲染流程等（收请求-视图渲染）。

```java
@ResponseBody
@RequestMapping("/async01")
public Callable<String> async01() {

  Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
      return "async01";
    }
  };

  return callable;
}
```



二、方式二：返回`DeferredResult`

一旦启用了异步请求处理功能 ，控制器就可以将返回值包装在`DeferredResult`，控制器可以从不同的线程异步产生返回值。优点就是可以实现两个完全不相干的线程间的通信。

以创建订单为例，客户端发起请求，应用A接受到这个请求，但应用A并不处理这个请求，通过消息中间件将这个请求交给应用B处理。

首先接受到请求后，创建一个`DeferredResult`对象，将这个对象保存起来，并将这个对象返回，这个请求就在等待中。当另外一个线程，例如消息中间件，调用了这个对象的`deferredResult.setResult()`方法。请求就会得到响应。 