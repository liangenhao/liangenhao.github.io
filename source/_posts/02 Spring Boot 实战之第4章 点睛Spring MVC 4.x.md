---
title: 02 Spring Boot 实战之第4章 点睛Spring MVC 4.x
date: 2018年3月6日
categories: Spring Boot 学习
tags: [Spring Boot, Spring MVC]
---

> 从纯Java配置的角度来搭建Spring MVC项目。

## 1 Spring MVC概述

一、【MVC和三层架构的关系】

- MVC：Model + View + Controller （数据模型 + 视图 + 控制器）

  > M：数据模型，包含数据的对象。
  >
  > V：视图页面，包含JSP，freeMarker,  Thymeleaf等
  >
  > C：控制器，注解了@Controller的类。

- 三层架构：Presentation tier + Application tier + Data tier （展现层 + 应用层 + 数据访问层）

  > 三层架构是整个应用的架构。

**MVC只存在于三层架构的展现层**。

<!--more-->

## 2 Spring MVC 项目搭建

Spring MVC 提供了一个`DispatcherServlet`来开发Web应用。在Servlet2.5及以下，需要配置`web.xml`。在Servlet3.0及以上，则不使用`web.xml`配置方式。通过**实现`WebApplicationInitilaizer`接口**来配置，等同于配置`web.xml`。

一、引入maven依赖：

```xml
<!-- Spring MVC -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>${spring-framework.version}</version>
</dependency>
<!-- 其他web依赖 -->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
  <version>${jstl.version}</version>
</dependency>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>${servlet.version}</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>javax.servlet.jsp</groupId>
  <artifactId>jsp-api</artifactId>
  <version>${jsp.version}</version>
  <scope>provided</scope>
</dependency>
```

二、在`src/main/resources/views`下新建`index.jsp`。

> 注意：将页面放在resources下是Spring Boot的习惯放置方式。

三、Spring MVC 配置：

```java
@Configuration
@EnableWebMvc// 1
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig {
  @Bean
  public InternalResourceViewResolver viewResolver() { // 2
    InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
    viewResolver.setPrefix("/WEB-INF/classes/views/");
    viewResolver.setSuffix(".jsp");
    viewResolver.setViewClass(JstlView.class);
    return viewResolver;
  }
}
```

> 1. `@EnableWebMvc`：开启一些默认的MVC配置。
> 2. 配置了一个JSP的视图解析器`ViewResolver`，用来映射路径和实际页面的位置。

四、Web配置：等价于配置`web.xml`配置文件。

```java
public class WebInitializer implements WebApplicationInitializer {//1

  @Override
  public void onStartup(ServletContext servletContext)
    throws ServletException {
    AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
    ctx.register(MyMvcConfig.class); // 注册配置类
    ctx.setServletContext(servletContext); //2

    Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx)); //3
    servlet.addMapping("/");
    servlet.setLoadOnStartup(1);
  }
}
```

> 1. **`WebApplicationInitializer`是Spring提供用来配置Servlet 3.0+配置的接口，从而替代了`web.xml`的配置。**
> 2. 新建`WebApplicationContext`，注册配置类，并将其和当前`servletContext`关联。
> 3. 注册Spring MVC的前端控制器`DispatcherServlet`。

五、编写控制器：

```java
@Controller
public class HelloController {
  @RequestMapping("/index")
  public  String hello(){
    return "index"; // 1
  }
}
```

> 1. 通过`MyMvcConfig`的Bean的配置，返回值为`index`，说明我们的页面放置的路径为`/WEB-INF/classes/views/index.jsp`。

六、运行：`http://localhost:8080/项目名/index`。访问的页面是index.jsp。

## 3  Spring MVC常用注解

一、【@Contorller】：

- 作用：表明这个类是Spring MVC里的Controller。
- 注意：在声明普通Bean时，使用`@Component`、`@Service`、`@Repository`、`@Controller`是等同的。**但在Spring MVC声明控制器Bean的时候，只能使用`@Controller`。**

二、【@RequestMapping】：

- 用来映射web请求。

三、【@ResponseBody】：

- 支持将返回值放在response体中，而不是返回一个页面。常用于ajax请求。
- 该注解可以放在返回值前或者方法上。

四、【@RequestBody】：

- 允许request的参数在request体中。将字符串转成对象。

- 该注解放在参数前。

- 条件：

  1. 请求必须是POST请求。

  2. 请求的Content-Type是`application/x-www-form-urlencoded`、`application/json`、`application/xml`都可以。

     > `application/json`、`application/xml`必须使用`@RequestBody`。​

五、【@PathVariable】：

- 用来接收路径参数。如`/content/{id}`。中的id，可以用注解来接受。

六、【@RestController】：

- 组合注解，组合了`@Controller`和`@ResponseBody`。


## 4 Spring MVC 基本配置

Spring MVC 的定制配置需要配置类继承一个`WebMvcConfigurerAdapter`类，并使用`@EnableWebMvc`注解，来开启对Spring MVC的配置支持。然后重写方法进行定制配置。

### 4.1 静态资源映射

静态文件需要直接访问，在配置里重写`addResourceHandlers`方法来实现：

```java
@Configuration
@EnableWebMvc // 1
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig extends WebMvcConfigurerAdapter {

  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/assets/**").addResourceLocations(
      "classpath:/assets/"); // 2
  }
}
```

> 1. **`@EnableWebMvc`开启Spring MVC支持**，若不加此注解，重写`WebMvcConfigurerAdapter`方法无效。
> 2. **`addResourceLocations`指的 是文件方式的目录。`addResourceHandler`指的是对外暴露的访问路径。**

### 4.2 拦截器配置

#### 4.2.1 步骤

步骤一：让普通的Bean实现`HandlerInterceptor`接口或者继承`HandlerInterceptorAdapter`类来实现自定义拦截器。

步骤二：重写`WebMvcConfigurerAdapter`的`addInterceptors`方法来注册自定义的拦截器。

#### 4.2.2 示例

一、编写自定义拦截器：

```java
public class DemoInterceptor extends HandlerInterceptorAdapter {//1

  @Override
  public boolean preHandle(HttpServletRequest request, //2
                           HttpServletResponse response, Object handler) throws Exception {
    long startTime = System.currentTimeMillis();
    request.setAttribute("startTime", startTime);
    return true;
  }

  @Override
  public void postHandle(HttpServletRequest request, //3
                         HttpServletResponse response, Object handler,
                         ModelAndView modelAndView) throws Exception {
    long startTime = (Long) request.getAttribute("startTime");
    request.removeAttribute("startTime");
    long endTime = System.currentTimeMillis();
    System.out.println("本次请求处理时间为:" + new Long(endTime - startTime)+"ms");
    request.setAttribute("handlingTime", endTime - startTime);
  }

}
```

> 1. 继承`HandlerInterceptorAdapter`类来实现自定义拦截器。
> 2. 重写`preHandle`方法，在请求发生前执行。
> 3. 重写`postHandle`方法，在请求完成后执行。

二、编写配置类：

```java
@Configuration
@EnableWebMvc 
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig extends WebMvcConfigurerAdapter {
  @Bean // 配置烂机器的Bean，或者在拦截器上用@Service
  public DemoInterceptor demoInterceptor() {
    return new DemoInterceptor();
  }

  @Override
  public void addInterceptors(InterceptorRegistry registry) {// 1
    registry.addInterceptor(demoInterceptor());
  }
}
```

> 1. 重写`addInterceptors`方法，注册拦截器。

### 4.3 快捷的ViewController

在`2 Spring MVC 项目搭建`一节中，我们配置页面跳转的方式是：

```java
@RequestMapping("/index")
public  String hello(){
  return "index"; 
}
```

这样没有任何的业务逻辑，只有简单的页面跳转，很麻烦。

我们可以通过在配置类中重写`addViewControllers`方法来简化配置：

```java
@Configuration
@EnableWebMvc 
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig extends WebMvcConfigurerAdapter {
  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/index").setViewName("/index");
  }
}
```

> `addViewController`：相当于`@RequestMapping`的请求路径。
>
> `setViewName`：相当于返回值，即视图名称。







> 笔记是在看《JavaEE开发的颠覆者 Spring Boot》一书学习Spring Boot 时，为了备忘记录的。大部分的内容来自此书，同时由于实操过程中由于版本高于书中的版本，Spring Boot中的API会有写变化，也会进行说明。
>
>  另外在其他书中或者博客中看到的一些内容，也会添加到笔记中备忘。

