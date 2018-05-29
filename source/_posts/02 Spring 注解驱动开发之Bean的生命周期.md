---
title: 0102 Spring 注解驱动开发之Bean的生命周期
date: 2018年5月29日
categories: Spring Boot
tags: [Spring Boot, Annotation]
---

Bean的生命周期：创建 -> 初始化 -> 销毁 的过程。Bean的生命周期由Spring 容器管理 。我们可以自定义Bean的初始化和销毁方法，容器在Bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法。

Bean创建：Bean创建的时候，单实例Bean是在容器启动的时候创建，多实例Bean是在每次获取的时候创建对象。

Bean初始化：对象创建完成，并赋值好后，调用初始化方法。

Bean销毁：单实例Bean是在Spring容器关闭的时候调用销毁的方法。**多实例Bean，Spring容器是不会调用销毁方法。**我们可以手动调用销毁方法。

<!-- more -->

由五种定义初始化和销毁方法的方式：

1. 方式一：在使用xml配置方式的时候，我们可以在`<bean>`标签中指定`init-method`和`destroy-method`两个属性来指定初始化和销毁方法，初始化和销毁方法不能有参数，但可以抛出异常。

2. 方式二：使用`@Bean`指定初始化和销毁方法。

3. 方式三：让Bean实现`InitializingBean`和`DisposableBean`接口

4. 方式四：使用JSR250规范里的`@PostConstruct`和`@PreDestroy`注解。

5. 方式五：使用Bean的后置处理器`BeanPostProcessor`。

   > 后置处理器的作用：在Bean初始化前后进行一些处理操作。

## 1 @Bean指定初始化和销毁方法

`@Bean`注解由两个属性：`initMethod`和`destroyMethod`。通过这两个方法来指定初始化和销毁方法。

一、有一个Bean如下：

```java
public class Car {

  public Car() {
    System.out.println("this is Car Constructor method");
  }

  public void init() {
    System.out.println("this is Car init method");
  }

  public void destroy() {
    System.out.println("this is Car destroy method");
  }
}
```

二、配置类如下，在配置类中我们注入了Car这个Bean，并使用`@Bean`注解指定了这个Bean的初始化和销毁方法。

```java
@Configuration
public class ManiConfigOfLifeCycle {
  @Bean(initMethod = "init", destroyMethod = "destroy")
  public Car car() {
    return new Car();
  }
}
```

## 2 Bean实现InitializingBean和DisposableBean接口

 让Bean实现`InitializingBean`接口，并重写初始化方法`afterPropertiesSet`。该方法会在Bean创建完成，并属性都赋值完成后调用。

让Bean实现`DisposableBean`接口，并重写销毁方法`destroy`。单实例Bean，该方法在容器关闭的时候调用。

示例：

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("this is Car constructor method");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("this is Car destroy method");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("this is Car init method");
    }
}
```

> 配置类中使用包扫描注入该Bean。

## 3 使用@PostConstruct和@PreDestroy注解

`@PostConstruct`：在Bean创建完成，并且属性赋值完成后，来执行初始化方法。

`@PreDestroy`：在容器销毁Bean之前，来执行销毁方法。

注意：`@PostConstruct`和`@PreDestroy`都是只能标注在方法上的注解。

示例：

```java
@Component
public class Dog {
  public Dog() {
    System.out.println("this is Dog constructor method");
  }

  @PostConstruct
  public void init() {
    System.out.println("this is Dog init method");
  }

  @PreDestroy
  public void destroy() {
    System.out.println("this is Dog destroy method");
  }

}
```

> 配置类中使用包扫描注入该Bean。

## 4、使用后置处理器BeanPostProcessor

### 4.1 BeanPostProcessor的示例

后置处理器`BeanPostProcessor`是一个接口，它有两个方法：`postProcessBeforeInitialization`和`postProcessAfterInitialization`。

- `postProcessBeforeInitialization`：在创建Bean实例，对Bean的属性赋值，**并在任何初始化方法调用之前执行**，例如实现`InitializingBean`接口的`afterPropertiesSet`方法，或者是自定义的`init-method`方法。

  > ```
  > 原文：Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean initialization callbacks (like InitializingBean's {@code afterPropertiesSet} or a custom init-method).
  > ```

- `postProcessAfterInitialization`：在初始化方法调用之后执行。

  > ```
  > 原文：Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean initialization callbacks (like InitializingBean's {@code afterPropertiesSet} or a custom init-method). 
  > ```

一、定义自己的后置处理器

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    System.out.println("postProcessBeforeInitialization" + bean + " " + beanName);
    // 返回值可以是原来的bean，也可以是一个包装的bean
    return bean;
  }

  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    System.out.println("postProcessAfterInitialization" + bean + " " + beanName);
    return bean;
  }
}
```

> 执行顺序：构造方法 -> 后置处理器的前置处理 -> afterPropertiesSet/@PostConstruct -> 后置处理器的后置处理。
>
> 简而言之，在 初始化方法执行之前执行`postProcessBeforeInitialization`，在初始化方法执行之后执行`postProcessAfterInitialization`。若没有自定义初始化方法，后置处理器也是会执行的。

### 4.2 BeanPostProcessor的原理

通过查看Spring源码，`AbstractAutowireCapableBeanFactory`类中有一个`initializeBean`方法，该方法中调用了`invokeInitMethods`方法，用来执行初始化方法(`afterPropertiesSet`或自定义初始化方法)。在调用了`invokeInitMethods`方法前调用了`applyBeanPostProcessorsBeforeInitialization`方法，在调用了`invokeInitMethods`方法后调用了`applyBeanPostProcessorsAfterInitialization`方法。

源码：

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
  if (System.getSecurityManager() != null) {
    AccessController.doPrivileged(new PrivilegedAction<Object>() {
      @Override
      public Object run() {
        invokeAwareMethods(beanName, bean);
        return null;
      }
    }, getAccessControlContext());
  }
  else {
    invokeAwareMethods(beanName, bean);
  }

  Object wrappedBean = bean;
  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  }

  try {
    invokeInitMethods(beanName, wrappedBean, mbd);
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
      (mbd != null ? mbd.getResourceDescription() : null),
      beanName, "Invocation of init method failed", ex);
  }

  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  }
  return wrappedBean;
}

// 后置处理器的前置处理
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
  throws BeansException {

  Object result = existingBean;
  // 遍历得到容器中的所有的BeanPostProcessor，挨个执行，如果一旦方法返回null，后面的BeanPostProcessor就不会执行了。
  for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
    result = beanProcessor.postProcessBeforeInitialization(result, beanName);
    if (result == null) {
      return result;
    }
  }
  return result;
}
```

> 在执行`initializeBean`之前执行了`populateBean`方法，对bean属性进行赋值。

### 4.3 BeanPostProcessor在Spring底层的使用

以几个`BeanPostProcessor`的实现类为例：

#### 4.3.1 ApplicationContextAwareProcessor

一、【作用】：是可以向组件中注入IOC容器。

二、【使用】：

例如有一个组件`Pig`，实现`ApplicationContextAware`接口，重写接口中的`setApplicationContext`方法。

```java
public class Pig implements ApplicationContextAware {

  private ApplicationContext applicationContext;

  @Override
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.applicationContext = applicationContext;
  }
}
```

这样就把容器注入到组件中了。

三、【原理】：

通过查看`ApplicationContextAwareProcessor`中的`postProcessBeforeInitialization`方法：

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor {
  public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
    AccessControlContext acc = null;
    if (System.getSecurityManager() != null && (bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware || bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware || bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
      acc = this.applicationContext.getBeanFactory().getAccessControlContext();
    }

    if (acc != null) {
      AccessController.doPrivileged(new PrivilegedAction<Object>() {
        public Object run() {
          ApplicationContextAwareProcessor.this.invokeAwareInterfaces(bean);
          return null;
        }
      }, acc);
    } else {
      this.invokeAwareInterfaces(bean);
    }

    return bean;
  }
}
private void invokeAwareInterfaces(Object bean) {
  if (bean instanceof Aware) {
    if (bean instanceof EnvironmentAware) {
      ((EnvironmentAware)bean).setEnvironment(this.applicationContext.getEnvironment());
    }

    if (bean instanceof EmbeddedValueResolverAware) {
      ((EmbeddedValueResolverAware)bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }

    if (bean instanceof ResourceLoaderAware) {
      ((ResourceLoaderAware)bean).setResourceLoader(this.applicationContext);
    }

    if (bean instanceof ApplicationEventPublisherAware) {
      ((ApplicationEventPublisherAware)bean).setApplicationEventPublisher(this.applicationContext);
    }

    if (bean instanceof MessageSourceAware) {
      ((MessageSourceAware)bean).setMessageSource(this.applicationContext);
    }

    if (bean instanceof ApplicationContextAware) {
      ((ApplicationContextAware)bean).setApplicationContext(this.applicationContext);
    }
  }

}
```

> 以我们这个为例，在bean执行初始化之前，先判断这个bean是否实现了`ApplicationContextAware`，如果是，则调用`invokeAwareInterfaces`，向bean中注入值。
>
> 如果是`ApplicationContextAware`，就将`applicationContext`注入。

#### 4.3.2 InitDestroyAnnotationBeanPostProcessor

一、【作用】：用来处理`@PostConstruct`和`@PreDestroy`注解的。这就是为什么标注了这两个注解的方法称为生命周期方法的原因。

二、【原理】：

```java
public class InitDestroyAnnotationBeanPostProcessor
  implements DestructionAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor, PriorityOrdered, Serializable {
  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
    try {
      metadata.invokeInitMethods(bean, beanName);
    }
    catch (InvocationTargetException ex) {
      throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
    }
    catch (Throwable ex) {
      throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
    }
    return bean;
  }
}
```

#### 4.3.3 AutowiredAnnotationBeanPostProcessor

【作用】：用来处理`@Autowired`注解，这就是为什么标注的`@AutoWired`注解的属性可以进行依赖注入的原因。