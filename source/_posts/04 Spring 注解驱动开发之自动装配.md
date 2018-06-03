---
title: 04 Spring 注解驱动开发之自动装配
date: 2018年6月3日
categories: Spring Boot
tags: [Spring Boot, Annotation]
---

何为自动装配：Spring 利用依赖注入（DI），完成对容器中对各个组件的依赖关系的赋值。

<!-- more -->

## 1 @Autowired、@Qualifier、@Primary

`@Autowired`：Spring 定义的注解。

1. 默认优先按照类型去容器中找对应的组件Bean。找到就赋值。
2. 如果找到多个相同类型的组件Bean，再将属性名称作为组件的id去容器中查找。
3. 有属性`required`，默认值为true，即必须的，意思为必须装配这个Bean。如果为false，非必须的，表示如果容器中没有这个Bean，就不装配。

`@Qualifier`：和`@Autowired`组合使用，用来指定需要装配的组件Bean的id，而不是使用属性名。

`@Primary`：让Spring进行自动装配的时候，默认使用首选的Bean：

```java
@Configuration
public class MainConfigOfAutowired {
    
    @Primary
    @Bean
    public PersonService personService() {
       return new PersonServiceImpl();
    }
}
```

> 若有多个类型相同的Bean，这样使用`@Autowired`自动装配的时候，默认使用的就是加了`@Primary`的Bean。若想执行其他的Bean就要使用`@Qualifier`。

## 2 @Resource、@Inject

`@Resource`和`@Inject`都是java规范的注解。

`@Resource`：默认按照组件Bean的名称自动装配。可以使用属性`name`执行组件的名称。

> 注意：`@Resource`不支持`@Primary`注解的功能。

`@Inject`：和`@Autowired`功能一样，同样支持`@Primary`注解的功能。

> 注意：`@Inject`需要额外引入`javax.inject`的依赖。

## 3 方法、构造器位置的自动装配

`@Autowired`注解不仅可以标注在属性上，还可以标注在构造器、参数、方法上。

一、【标注在方法位置】：`@Autowired`标注在方法上时，容器创建对象时，就会调用方法完成赋值。方法使用的参数，自定义类型的值会从容器中获取

```java
@Autowired
public void setCar(Car car) { // Car对象从容器中获取
  this.car = car;
}
```

二、【标注在构造器位置】：

 容器加载组件时，默认会调用无参构造器创建对象。

若在有参构造器上标注`@Autowired`注解，创建对象时会调用该有参构造器，构造器的参数从容器中获取。 

```java
@Autowired
public Person(Car car) {
  this.car =  car;
}
```

注意：如果组件Bean只有唯一的一个有参构造器，该构造器的`@Autowired`可以省略。参数还是可以从容器中获取。

三、【标注在参数位置】：

```java
public void setCar(@Autowired Car car) { // Car对象从容器中获取
  this.car = car;
}

public Person(@Autowired Car car) {
  this.car =  car;
}
```

> 标注在参数上和标注在方法上效果一样。

如果采用`@Bean`的方式注入Bean，有参构造的参数该怎么赋值：

```java
@Bean
public Person person(Car car) {
  return new Person(car);
}
```

可以注解在注入Bean的方法的形参上加上构造的参数。

## 4 Aware注入Spring底层组件

如果我们自己定义的组件Bean想使用Spring容器底层的一些组件，例如：`ApplicationContext`、`BeanFactory`等等。可以让自定义的组件实现`XxxAware`接口，就可以将Spring容器的一些组件注入到我们自定义的组件中。 

示例：

```java
public class Pig implements ApplicationContextAware, BeanNameAware,EmbeddedValueResolverAware {

  private ApplicationContext applicationContext;

  @Override
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.applicationContext = applicationContext;
  }
  @Override
  public void setBeanName(String name) {

  }
  @Override
  public void setEmbeddedValueResolver(StringValueResolver resolver) {

  }
}
```

> 实现`ApplicationContextAware`接口并重写`setApplicationContext`方法，就可以将`ApplicationContext`对象传入组件Bean中。
>
> 实现`BeanNameAware`接口并重写`setBeanName`方法，就可以获取到容器为当前Bean起的名称。
>
> 实现`EmbeddedValueResolverAware`接口并重写`setEmbeddedValueResolver`方法，就可以获取到`StringValueResolver`值解析器，就可以用来解析`#{}`和`${}`。

`XxxAware`的原理是由`XxxProcessor`后置处理器来实现的，每个`Aware`都会对应一个后置处理器。

## 5 @Profile

Profile：可以根据当前环境，动态的激活或切换一系列组件的功能。

以数据源为例，在开发环境、测试环境和生产环境中，数据源的配置都不一样，在不更改代码的情况下，动态的切换这三种环境，就需要使用Profile。

`@Profile`：指定组件在哪个环境的情况下才能被注册到容器中。默认时default。

一、在配置类中注入三种不同环境的数据源

```java
@Configuration
public class MainConfigOfProfile {

    @Profile("test")
    @Bean
    public DataSource dataSourceTest() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/springtest");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    @Profile("dev")
    @Bean
    public DataSource dataSourceDev() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/crm");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    @Profile("prod")
    @Bean
    public DataSource dataSourceProd() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/leh_test");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
}
```

> 使用`@Profile`注解指定在哪种环境下会被注入。

二、指定不同的环境

方式一：使用命令行动态参数：`-Dspring.profiles.active=test`

方式二：代码的方式激活：

```java
// 创建applicationContext对象
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
// 设置需要激活的环境
applicationContext.getEnvironment().setActiveProfiles("test");
// 注册配置类
applicationContext.register(MainConfigOfProfile.class);
// 启动刷新容器
applicationContext.refresh();
```

`@Profile`不仅可以写在方法上，还可以写在类上，写在类上，只有指定的环境，整个配置类里的所有配置才会生效。

没有标注`@Profile`的bean在任何环境都会加载。