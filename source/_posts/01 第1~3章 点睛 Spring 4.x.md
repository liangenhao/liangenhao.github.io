---
title: 01 第1~3 章 点睛 Spring 4.x
date: 2018年3月4日 18点50分
categories: Spring Boot 学习
tags: [Spring Boot, 依赖注入, AOP]
---

> 笔记是在看《JavaEE开发的颠覆者 Spring Boot》一书学习Spring Boot 时，为了备忘记录的。大部分的内容来自此书，同时由于实操过程中由于版本高于书中的版本，Spring Boot中的API会有写变化，也会进行说明。
>
> 另外在其他书中或者博客中看到的一些内容，也会添加到笔记中备忘。

## 1 Spring 的基础配置

> 由于不是第一次接触Spring 框架，所以不对Spring 和 Maven 的部分做过多的摘录。

> 这3章主要对Spring的一些重要功能进行点睛和了解一些简单的java配置。因为Spring Boot的所有配置都是通过Java配置呈现的，而不是xml配置。

### 1.1 依赖注入

两个重要概念 控制反转（IOC）和依赖注入（DI）：

- 控制反转（IOC）：是一种设计思想，将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。
- 依赖注入（DI）：组件之间依赖关系由容器在运行期决定。即由容器动态的将某个依赖关系注入到组件之中。

<!-- more -->

> 控制反转和依赖注入其实是同一概念的不同角度的描述。

> 从CSDN上找到的对IOC和DI 非常好的解释，上面的解释也是从其中摘抄 - [依赖注入和控制反转的理解，写的太好了。](http://blog.csdn.net/bestone0213/article/details/47424255)

依赖注入的主要目的是为了解耦，体现了一种“组合”的理念。

Spring 提供使用xml、注解、Java配置实现Bean的创建和注入：

声明Bean的注解：

- `@Component`：没有明确的角色。

- `@Service`：在service层使用。

- `@Repository`：在dao层使用。

- `@Controller`：在展现层使用（Spring MVC）使用。

  > `@Component`、`@Service`、`@Repository`三个注解目前没有区别，只是用来区分不同的层。
  >
  > **Spring MVC 声明Bean的时候只能使用`@Controller`。**

注入Bean的注解，一般情况下通用：

- `@Autowired`：Spring提供的注解。

- `@Inject`：JDR-330提供的注解。

- `@Resource`：JDR-250提供的注解。

  > 以上三个注解可以注解在set方法或者属性上。

### 1.2 Java 配置

Java 配置时Spring 4.x 推荐的配置方式，可以完全替代xml配置。

Java 配置通过`@Configuration`和`@Bean`来实现的。

- `@Configuration`：声明当前类是一个配置类，相当于一个Spring的xml配置文件。

- `@Bean`注解在方法上，声明当前方法的返回值为一个Bean。

  > 注意：使用`@Bean`和使用`@Service`等注解是声明Bean的两种不同方式。使用其中一种即可，不要对同一个Bean 同时使用这两种方式。

> 我们通常使用Java配置和注解混合配置的方式。**原则是：全局配置使用Java配置（如数据库相关配置，MVC相关配置）、业务Bean的配置使用注解配置（`@Component`、`@Service`、`@Repository`、`@Controller`）。**

示例：

1. 编写两个Bean：这两个Bean都没有使用注解配置。

   ```java
   public class FunctionService {	
   	public String sayHello(String word){
   		return "Hello " + word +" !"; 
   	}
   }
   public class UseFunctionService {
   	private FunctionService functionService;
   	public void setFunctionService(FunctionService functionService) {
   		this.functionService = functionService;
   	}
   	public String SayHello(String word){
   		return functionService.sayHello(word);
   	}
   }
   ```

2. 编写配置类：

   ```java
   @Configuration // 1
   @ComponentScan("需要扫描的包") // 2
   public class JavaConfig {
     @Bean // 3
     public FunctionService functionService(){
       return new FunctionService();
     }

     @Bean
     public UseFunctionService useFunctionService(){
       UseFunctionService useFunctionService = new UseFunctionService();
       useFunctionService.setFunctionService(functionService()); //4
       return useFunctionService;

     }

     // @Bean
     // public UseFunctionService useFunctionService(FunctionService functionService){//5
     // 	UseFunctionService useFunctionService = new UseFunctionService();
     // 	useFunctionService.setFunctionService(functionService);
     // 	return useFunctionService;
     // }
   }
   ```

   > 1. `@Configuration`注解声明当前类是一个配置类。
   > 2. `@ComponentScan` 如果使用了`@Service`、`@Component`、`@Repository`、`@Controller`等注解，需要使用包扫描注解。
   > 3. `@Bean`声明当前方法的返回值是一个Bean。**Bean的名称就是方法名。**
   > 4. 注入`FunctionService`的Bean时候直接调用`functionService()`。
   > 5. 另一种注入方式，直接将`functionService`作为参数给`useFunctionService()`。

### 1.3 AOP

Spring 的AOP的存在时为了解耦。AOP可以让一组类共享相同的行为。

Spring 支持 AspectJ 的注解式切面编程。

1. 使用`@Aspect`声明一个切面。
2. 使用`@After`、`@Before`、`@Around`定义通知（advice），可直接将拦截规则（切点）作为参数。
3. 其中`@After`、`@Before`、`@Around`参数的拦截规则为切点（PointCut），为了使切点复用，可使用`@PointCut`专门定义拦截规则，然后在`@After`、`@Before`、`@Around`的参数中调用。
4. 其中符合条件的每一个被拦截处为连接点（JoinPoint）。

#### 1.3.1 示例

> 示例将演示**基于注解拦截**和**基于方法规则拦截**两种方式，演示一种模拟记录操作的日志系统的实现。
>
> - 基于注解的拦截：**自定义一个注解**，对标注了该注解的方法进行拦截。可以更好的控制要拦截的粒度和获得更丰富的信息。
> - 基于方法规则的拦截：通过写**切点表达式**，对表达式所解析的方法进行拦截。

1. 添加 Spring 的 AOP 和 AspectJ 的依赖：

   ```xml
   <!-- spring aop支持 -->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>${spring-framework.version}</version>
   </dependency>
   <!-- aspectj支持 -->
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjrt</artifactId>
       <version>1.8.6</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.8.5</version>
   </dependency>
   ```

2. 基于注解的拦截：

   1. 自定义拦截规则的注解：

      ```java
      @Target(ElementType.METHOD)
      @Retention(RetentionPolicy.RUNTIME)
      @Documented
      public @interface Action {
          String name();
      }
      ```

      > 注意：注解本身是没有功能的，是一种元数据，元数据即解释数据的数据，这就是所谓配置。

   2. 编写使用注解的被拦截类：

      ```java
      @Service
      public class DemoAnnotationService {
      	@Action(name="注解式拦截的add操作")
          public void add(){} 
      }
      ```

   3. 编写切面：

      ```java
      @Aspect //1
      @Component 
      public class LogAspect {
      	
        @Pointcut("@annotation(com.wisely.highlight_spring4.ch1.aop.Action)") //2
        public void annotationPointCut(){};

        @After("annotationPointCut()") //3
        public void after(JoinPoint joinPoint) {
          MethodSignature signature = (MethodSignature) joinPoint.getSignature();
          Method method = signature.getMethod();
          Action action = method.getAnnotation(Action.class); 
          System.out.println("注解式拦截 " + action.name()); //4
        }
      }
      ```

      > 1. 通过`@Aspect`注解声明一个切面。
      > 2. 通过`@Pointcut`注解声明切点，表明拦截标注了`@Action`注解的方法。
      > 3. 通过`@After`注解声明一个通知，并使用`@PointCut`定义的切点。
      > 4. 通过反射可获得注解上的属性。

3. 基于方法规则的拦截：

   1. 编写使用方法规则被拦截类：

      ```java
      @Service
      public class DemoMethodService {
      	public void add(){}
      }
      ```

   2. 编写切面：

      ```java
      @Aspect 
      @Component 
      public class LogAspect {
      	  
        @Before("execution(* com.wisely.highlight_spring4.ch1.aop.DemoMethodService.*(..))") //1
        public void before(JoinPoint joinPoint){
          MethodSignature signature = (MethodSignature) joinPoint.getSignature();
          Method method = signature.getMethod();
          System.out.println("方法规则式拦截,"+method.getName());
        }
      	
      }
      ```

      > 通过`@Before`注解声明一个通知，此通知直接使用拦截规则（切点表达式）作为参数。
      >
      > 也可以使用`@PointCut`注解来编写切点表达式，然后在`@Before`注解里引用切点。这样提高重用性：
      >
      > ```java
      > @Aspect 
      > @Component 
      > public class LogAspect {
      >   
      >   @Pointcut("execution(* com.wisely.highlight_spring4.ch1.aop.DemoMethodService.*(..))")
      >   public void pointCut(){};
      >
      >   @Before("pointCut()") 
      >   public void before(JoinPoint joinPoint){
      >     MethodSignature signature = (MethodSignature) joinPoint.getSignature();
      >     Method method = signature.getMethod();
      >     System.out.println("方法规则式拦截,"+method.getName());
      >   }
      > 	
      > }
      > ```

4. 编写配置类：

   ```java
   @Configuration
   @ComponentScan("com.wisely.highlight_spring4.ch1.aop")
   @EnableAspectJAutoProxy //1
   public class AopConfig {

   }
   ```

   > 使用`@EnableAspectJAutoProxy`注解开启Spring 对 AspectJ 代理的支持。

## 2 Spring 常用配置

### 2.1 Bean的Scope

Scope 是来描述Spring容器如何新建Bean的实例的。也可以说表示Bean的作用域。通过`@Scope`注解来实现。

`@Scope`的value属性有以下几个值：

- `Singleton`（默认）：单例模式。一个Spring容器中只有一个Bean的实例。此为Spring的默认配置，全容器共享一个实例。
- `Prototype`：每次调用新建一个Bean的实例。
- `Request`：Web项目中，给每一个http request 新建一个Bean实例。
- `Session`：Web项目中，给每一个http session 新建一个Bean实例。
- `GlobalSession`：这个只在portal应用中有用，给每一个 global http session 新建一个Bean实例。

### 2.2 Spring EL 和 资源调用

可以使用SpEL表达式实现普通字符，操作系统属性，运算结果，其他Bean的属性，文件内容，网址内容，属性文件的注入。

示例：

新建`test.properties`属性文件：

```properties
book.author=wangyunfei
book.name=spring boot
```

使用`@Value`的SpEL表达式。

```java
@Service
@Getter
@Setter
public class DemoService {
  @Value("demo service") // 注入普通字符串
  private String another;
}


@Configuration
@ComponentScan("com.wisely.highlight_spring4.ch2.el")
@PropertySource("classpath:com/wisely/highlight_spring4/ch2/el/test.properties")// 1
public class ElConfig {
	
  @Value("I Love You!") // 注入普通字符串
  private String normal;

  @Value("#{systemProperties['os.name']}") // 注入操作系统属性
  private String osName;

  @Value("#{ T(java.lang.Math).random() * 100.0 }") // 注入表达式结果
  private double randomNumber;

  @Value("#{demoService.another}") // 注入其他Bean的属性
  private String fromAnother;

  @Value("classpath:com/wisely/highlight_spring4/ch2/el/test.txt") // 注入文件资源
  private Resource testFile;

  @Value("http://www.baidu.com") // 注入网址资源
  private Resource testUrl;

  @Value("${book.name}") // 1 注入属性配置文件
  private String bookName;

  @Autowired
  private Environment environment; // 2 注入属性配置文件

  @Bean // 1 声明Bean
  public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
    return new PropertySourcesPlaceholderConfigurer();
  }
	
  public void outputResource() {
    try {
      System.out.println(normal);
      System.out.println(osName);
      System.out.println(randomNumber);
      System.out.println(fromAnother);

      System.out.println(IOUtils.toString(testFile.getInputStream()));
      System.out.println(IOUtils.toString(testUrl.getInputStream()));
      System.out.println(bookName);
      System.out.println(environment.getProperty("book.author"));
    } catch (Exception e) {
      e.printStackTrace();
    }

  }
	
}
```

> 1. 注入配置文件需要使用`@PropertySource`注解指定属性文件的位置。使用`@Value`注入，需要配置一个`PropertySourcesPlaceholderConfigurer`的Bean。
>
>    配置这个类等价于xml配置中的`<context:property-placeholder location="classpath:application.properties"/>`
>
>    > **注意：注入属性配置文件的表达式使用`$`而不是`#`。**
>
> 2. 注入属性配置文件还可以使用`Environment`。使用`getProperty()`方法获取属性。

### 2.3 Bean 的初始化和销毁

有两种方式：

一、【Java配置方式】：

使用`@Bean`的`initMethod`和`destroyMethod`方法。相当于xml配置的`init-method`和`destroy-method`。

1. 编写Bean

   ```java
   public class BeanWayService {
     public void init(){
       System.out.println("@Bean-init-method");
     }
     public BeanWayService() {
       super();
       System.out.println("初始化构造函数-BeanWayService");
     }
     public void destroy(){
       System.out.println("@Bean-destory-method");
     }
   }
   ```

2. 编写配置类：

   ```java
   @Configuration
   @ComponentScan("com.wisely.highlight_spring4.ch2.prepost")
   public class PrePostConfig {
   	
     @Bean(initMethod="init",destroyMethod="destroy") // 1
     BeanWayService beanWayService(){
       return new BeanWayService();
     }

   }
   ```

   > 1. `initMethod`和`destroyMethod`指定`BeanWayService`类的init和destroy方法在构造之后、Bean销毁之前执行。

二、【注解方式】：

使用JSR-250的`@PostConstruct`和`@PreDestroy`注解。

> JDK8 不需要引入`jsr-250`的依赖。

1. 编写Bean。

   ```java
   public class JSR250WayService {
     @PostConstruct //1
     public void init(){
       System.out.println("jsr250-init-method");
     }
     public JSR250WayService() {
       super();
       System.out.println("初始化构造方法-JSR250WayService");
     }
     @PreDestroy //2
     public void destroy(){
       System.out.println("jsr250-destory-method");
     }

   }
   ```

   > 1. **`@PostConstruct`注解，在构造函数执行完之后执行。**
   > 2. **`@PreDestroy`注解，在Bean销毁之前执行。**

2. 编写配置类：

   ```java
   @Configuration
   @ComponentScan("com.wisely.highlight_spring4.ch2.prepost")
   public class PrePostConfig {
     @Bean
     JSR250WayService jsr250WayService(){
       return new JSR250WayService();
     }
   }
   ```


### 2.4 Profile

Profile 为在**不同环境下使用不同的配置**提供了支持。

1. 通过设定`ActiveProfiles`来设定当前context需要使用的配置环境。在开发中使用`@Profile`注解类或者方法，达到在不同情况下选择实例化不同的Bean。

一、【非web项目】：

```java
@Getter
@Setter
public class DemoBean {
  private String content;

  public DemoBean(String content) {
    this.content = content;
  }
}
```

Profile 配置：

```java
@Configuration
public class ProfileConfig {
  @Bean
  @Profile("dev")
  public DemoBean devDemoBean() {
    return new DemoBean("from development profile");
  }
  @Bean
  @Profile("prod")
  public DemoBean devDemoBean() {
    return new DemoBean("from production profile");
  }
}
```

运行：

```java
public class Main {
  public static void main(String[] args) {
    AnnotationConfigApplicationContext context =  new AnnotationConfigApplicationContext();

    context.getEnvironment().setActiveProfiles("dev"); //1
    context.register(ProfileConfig.class);//2
    context.refresh(); //3

    DemoBean demoBean = context.getBean(DemoBean.class);

    System.out.println(demoBean.getContent());

    context.close();
  }
}
```

> 1. 将活动的Profile设置为prod。
> 2. 后注册Bean配置类。
> 3. 刷新容器。

二、【web项目】：

Web项目需要设置Servlet的context parameter 中。

例如在Spring MVC中的前端控制器`DispatcherServlet`中配置：

```xml
<filter>   
    <filter-name>springSecurityFilterChain</filter-name>    
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  <!-- 激活profile -->
  <init-param>
  	<param-name>spring.profiles.active</param-name>
    <param-value>production</param-value>
  </init-param>
</filter>  
<filter-mapping>  
    <filter-name>springSecurityFilterChain</filter-name>  
    <url-pattern>/*</url-pattern>    
</filter-mapping> 
```

