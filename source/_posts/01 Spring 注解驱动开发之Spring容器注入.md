---
title: 01 Spring 注解驱动开发之Spring容器注入
date: 2018-05-27
categories: Spring Boot
tags: [Spring Boot, Annotation]
---

在我们使用传统的 Spring MVC + Spring + Mybatis 整合开发时，通常采用的是使用xml配置 + Java注解混合式的开发，即跟业务相关的使用Java注解，配置相关的使用xml文件。这就造成了我们需要写大量的xml配置代码。

在微服务兴起后，Spring Boot 和 Spring Cloud 等微服务框架都摒弃了xml配置文件的配置方式，大量使用Java注解来进行开发。

Spring 注解驱动开发分为Spring 容器相关，Spring 原理，Spring MVC。

首先从Spring容器说起，Spring容器的两个重要概念就是控制反转和依赖注入。Spring将所有的组件都放在了容器中，组件中的关系通过容器来进行自动装配。那如何使用注解的形式进行容器的注入和自动装配？

<!-- more -->



## 1 组件注册

### 1.1 注入Bean：@Configuration、@Bean

#### 1.1.1 xml配置方式

最原始的配置方式，需要将注入的Bean全都配置到Spring的配置文件中。

首先，我们有一个bean对象。

```java
public class Persion {
  private String name;
  private Integer age;
  // getter,setter,construction,toString
}
```

要想将这个Bean对象注入到Spring容器中，如下：

一、在Spring的配置文件 `applicationContext.xml` 配置文件中配置bean：

```xml
<bean id="person" class="com.enhao.spring.annotation.bean.Person">
  <property name="name" value="enhao"/>
  <property name="age" value="18"/>
</bean>
```

二、测试

```java
public static void main(String[] args) {
  ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
  Person person = (Person) applicationContext.getBean("person");
  System.out.println(person);
}
```

> 使用`ClassPathXmlApplicationContext`读取配置文件。

#### 1.1.2 注解驱动配置方式

注解驱动开发将不再使用`applicationContext.xml`xml配置文件，使用Java类取代配置文件。

一、**配置类等同于之前的配置文件**。在配置类上使用`@Configuration`注解，表明这是一个配置类。

注入对象时，使用`@Bean`注解，给容器中注册一个Bean。**方法返回值为Bean的类型，方法名为Bean的id。**也可以使用`@Bean`注解的value属性设置id。

```java
// 配置类 == 配置文件
@Configuration
public class MainConfig  {

    @Bean
    public Person person() {
        return new Person("enhao", 18);
    }
}
```

> `@Bean`注解，等价于1.1中的`<bean></bean>`配置方式。

二、测试：

```java
public static void main(String[] args) {
  ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
  Person person = (Person) applicationContext.getBean("person");
  System.out.println(person);
}
```

> 使用`AnnotationConfigApplicationContext`读取配置类。不再是`ClassPathXmlApplicationContext`了。

### 1.2 包扫描：@ComponentScan

#### 1.2.1 xml + 注解混合配置方式

在Java5引入注解的特性后，诞生了xml + 注解混合式开发的方式。

一、在Spring的配置文件 `applicationContext.xml` 配置文件中开启注解扫描：

```xml
<context:component-scan base-package="com.enhao.spring.annotation.service"/>
```

> 主要标注了`@controller`、`@Service`、`@Repository`、`@Component`注解中任意一个，都会被扫描自动注入容器中。

二、在需要注入的Bean类上添加`@Component`注解：

```java
@Service("personService")
public class PersonServiceImpl implements PersonService {
    @Override
    public void show() {
        System.out.println("this is personServiceImpl bean");
    }
}
```

三、测试：

```java
public static void main(String[] args) {
  ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
  PersonService personService = (PersonService) applicationContext.getBean("personService");
  personService.show();
}
```

#### 1.2.2 注解驱动配置方式

一、注解驱动的配置包扫描的方式是直接在配置类上使用`@ComponentScan`注解即可。

```java
@Configuration
@ComponentScan({"com.enhao.spring.annotation"})
public class MainConfig  {
  
}
```

二、在需要注入的Bean类上添加`@controller`、`@Service`、`@Repository`、`@Component`注解中任意一个即可。

`@ComponentScan`的属性：

- `value`：指定要扫描的包。
- `excludeFilters`：指定扫描的时候按照什么规则排除哪些组件。值是`Filter[]`。
  - `@Filter`注解：`type`属性表示指定排除的规则：**按照注解排除，按照AspectJ表达式排除，按照给定的类型排除、按照自定义排除和按照正则排除**；`classes`属性表示根据排除规则需要排除的类。
- `includeFilters`：指定扫描的时候只需要包含哪些组件。值为`Filter[]`。注意：如果使用`includeFilters`，必须关闭默认的过滤规则(默认过滤规则是包含所有)，将`useDefaultFilters`设置为false。

```java
@ComponentScan(value = "com.enhao.spring.annotation", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
})
```

> `FilterType.ANNOTATION`使用按照注解排除的规则，排除标注了`@Controller`和`@Service`注解的Bean。

```java
@ComponentScan(value = "com.enhao.spring.annotation", useDefaultFilters = false, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
})
```

> 使用按照注解的规则，只包含使用了`@Controller`注解的Bean。
>
> 注意：使用`includeFilters`时，必须将`useDefaultFilters`的值设置为false。

```java
@ComponentScan(value = "com.enhao.spring.annotation", useDefaultFilters = false, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {PersonService.class})
})
```

> `FilterType.ASSIGNABLE_TYPE`包含给定的类型，这里会包含是`PersonService`类型的Bean。



由于java8引入了重复注解的概念（具体可以查看Java8新特性），所有可以写多个`@ComponentScan`注解来指定不同的规则。

如果是Java8以前的版本，可以使用`@ComponentScans`注解，该注解里可以写多个`@ComponentScan`注解。

### 1.3 作用域：@Scope

#### 1.3.1 @Scope和@Bean一起使用

Spring 的Bean默认都是单实例的。可用使用`@Scope`来指定Bean的作用域。

通过`@Scope`注解的`value`属性来指定Bean的作用域，值有四种：`singleton`(默认值)、`prototype`、`request`、`session`。

```java
@Configuration
public class ScopeConfig {
    @Bean
    @Scope("prototype")
    public Person person2() {
        return new Person("enhao", 19);
    }
}
```

> 等同于`<bean scope=""></bean>`

注意：

在默认单实例时，在Spring容器启动时就会调用方法将Bean对象放到容器中。以后每次获取都是直接从容器中获取的。

在多实例时，Spring 容器启动时并不会区创建多实例Bean的对象。多实例Bean对象是在调用的时候才放到容器中的，而且是每调用一次就向容器中放入一个对象。

#### 1.3.2 @Scope和@Component一起使用

在使用包扫描的时候，如果要指定Bean的作用域，可以和`@Component`、`@Service`、`@Repository`、`@Controller`等注解一起使用，使用方式和上面一样。

### 1.4 懒加载：@Lazy

> **懒加载只针对于单实例的Bean**。因为单实例的Bean是在容器启动的时候就加载的。

懒加载：容器启动的时候不创建对象。在第一次使用（获取）Bean的时候创建对象，并初始化。

#### 1.4.1 @Lazy和@Bean一起使用

```java
@Configuration
public class ScopeConfig {
  @Bean
  @Lazy
  public Person person2() {
    return new Person("enhao", 19);
  }
}
```

#### 1.4.2 @Lazy和@Component一起使用

在使用包扫描的时候，如果要指定Bean的懒加载，可以和`@Component`、`@Service`、`@Repository`、`@Controller`等注解一起使用，使用方式和上面一样。

### 1.5 条件注解：@Conditional

作用：按照一定的条件进行判断，满足条件给容器注册Bean。

`@Conditional`注解的` value`属性是继承了`Condition`的Class对象。因此需要实现`Condition`接口。

例子：判断当前的系统，如果是linux系统，则创建Persion3对象。如果是Windows系统则创建Person4对象。

```java
// 判断是否是windows系统
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String osName = environment.getProperty("os.name");
        if (osName.contains("Windows")) {
            return true;
        }
        return false;
    }
}
// 判断是否是linux系统
public class LinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String osName = environment.getProperty("os.name");
        if (osName.contains("linux")) {
            return true;
        }
        return false;
    }
}
// 配置类
@Configuration
public class ScopeConfig {

    @Conditional(LinuxCondition.class)
    @Bean
    public Person person3() {
        return new Person("enhao1", 20);
    }

    @Conditional(WindowsCondition.class)
    @Bean
    public Person person4() {
        return new Person("enhao2", 21);
    }
}
```

### 1.6 快速注册组件：@Import

#### 1.6.1 直接导入配置类或普通类

@Import注解支持导入配置类，也支持导入普通的java类，并将其声明成一个bean。

```java
@Configuration
@Import({Color.class})
public class MainConfig2 {
}
```

> 注入Bean的id默认是组件的全类名。

#### 1.6.2 使用ImportSelector

使用`ImportSelector`导入选择器接口。接口里的`selectImports`方法的返回值**返回的是需要导入的组件的全类名数组。**

一、自定义导入选择器：

```java
public class MyImportSelector implements ImportSelector {

  // 参数AnnotationMetadata：当前标注@Import注解的类的所有注解信息
  @Override
  public String[] selectImports(AnnotationMetadata annotationMetadata) {

    // 返回值不要返回null，否则会报空指针异常。
    return new String[]{"com.enhao.spring.annotation.bean.Blue",
                        "com.enhao.spring.annotation.bean.Yellow"};
  }
}
```

二、在`@Import`注解中引入自定义的导入选择器。

```java
@Configuration
@Import({Color.class, MyImportSelector.class})
public class MainConfig2 {
}
```

#### 1.6.3 使用ImportBeanDefinitionRegistrar

使用`ImportBeanDefinitionRegistrar`接口，实现`registerBeanDefinitions`方法，在容器中自己添加组件。

一、自定义一个`ImportBeanDefinitionRegistrar`实现类：

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
  /**
     * 把所需要添加到容器中的bean，通过BeanDefinitionRegistry注册进来
     * @param annotationMetadata 当前标注@Import注解的类的所有注解信息
     * @param beanDefinitionRegistry BeanDefinitionRegistry Bean定义的注册类
     */
  @Override
  public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
    boolean blue = beanDefinitionRegistry.containsBeanDefinition("com.enhao.spring.annotation.bean.Blue");
    boolean yellow = beanDefinitionRegistry.containsBeanDefinition("com.enhao.spring.annotation.bean.Yellow");
    if (blue && yellow) {
      RootBeanDefinition beanDefinition = new RootBeanDefinition(Rainbow.class);
      // 注册bean
      beanDefinitionRegistry.registerBeanDefinition("rainbow", beanDefinition);
    }

  }
}
```

二、在`@Import`中使用

```java
@Configuration
@Import({MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
public class ScopeConfig {}
```

### 1.7 使用FactoryBean注册组件

一、自定义一个FactoryBean实现类：

```java
public class ColorFactoryBean implements FactoryBean<Color> {
  /**
       *
       * @return 返回一个Color对象，这个对象会添加到容器中
       * @throws Exception
       */
  @Override
  public Color getObject() throws Exception {
    System.out.println("this is a color factory bean");
    return new Color();
  }

  /**
     *
     * @return 返回对象类型
     */
  @Override
  public Class<?> getObjectType() {
    return Color.class;
  }

  /**
     * 是单例吗
     * @return true - 是单例模式 false - 是多例模式
     */
  @Override
  public boolean isSingleton() {
    return true;
  }
}
```

二、在配置类中配置工厂类Bean：

```java
@Bean
public ColorFactoryBean colorFactoryBean() {
  return new ColorFactoryBean();
}
```

> 默认获取到的是工厂Bean调用`GetObject`创建的对象，这里是`Color`。
>
> 要获取工厂Bean的本身，我们需要给id前面加一个`&`标识。