---
title: 05 Spring 注解驱动开发之AOP
date: 2018-06-10
categories: Spring Boot
tags: [Spring Boot, Annotation, AOP]
---

<!-- more -->

## 1 AOP 示例

 示例：编写切面用来记录日志。

```java
@Component
@Aspect
public class LogAspects {

  //抽取公共的切入点表达式
  //1、本类引用
  //2、其他的切面引用
  @Pointcut("execution(com.enhao.spring.annotation.service.*.*(..))")
  public void pointCut(){};

  //@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
  @Before("pointCut()")
  public void logStart(JoinPoint joinPoint){
    Object[] args = joinPoint.getArgs();
    System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
  }

  @After("com.atguigu.aop.LogAspects.pointCut()")
  public void logEnd(JoinPoint joinPoint){
    System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
  }

  //JoinPoint一定要出现在参数表的第一位
  @AfterReturning(value="pointCut()",returning="result")
  public void logReturn(JoinPoint joinPoint,Object result){
    System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
  }

  @AfterThrowing(value="pointCut()",throwing="exception")
  public void logException(JoinPoint joinPoint,Exception exception){
    System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
  }
}
```

> `@Aspect`注解告知标注的类是切面类。

**开启基于注解的AOP模式：**（这个一定要加的）

一、在xml配置方式中：

```xml
<aop:aspectj-autoproxy />
```

二、使用注解的配置方式，在配置类上标注`@EnableAspectJAutoProxy`注解。

```java
@Configuration
@EnableAspectJAutoProxy
public class MainConfigOfAOP {
}
```

## 2 声明式事务示例

一、【需要导入的相关依赖】：

包括：数据源、数据库驱动、Spring-jdbc。

二、【配置数据源】：

JdbcTemplate：Spring提供的简化数据库操作的工具。

```java
@Configuration
public class TxConfig {

  @Bean
  public DataSource dataSource() throws PropertyVetoException {
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setUser("root");
    dataSource.setPassword("root");
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
    return dataSource;
  }

  @Bean
  public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
    return jdbcTemplate;
  }
}
```

三、【编写Dao和Service】：

```java
@Repository
public class PersonDaoImpl implements PersonDao {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  @Override
  public void insert() {
    String sql = "insert into person (name, age) value (?, ?)";
    jdbcTemplate.update(sql, "enhao", 18);
  }
}

@Service("personService")
public class PersonServiceImpl implements PersonService {

  @Autowired
  private PersonDao personDao;

  @Transactional
  @Override
  public void insert() {
    personDao.insert();
  }
}
```

> 在Service层给方法加上事务。

四、【开启注解的事务支持】：

在配置类上标注：`@EnableTransactionManagement`。等价于xml配置：`<tx:annotation-driven />`。

在配置类中配置事务管理器transactionManager：

```java
@EnableTransactionManagement // 开启注解事务支持
@ComponentScan("com.enhao.spring.annotation")
@Configuration
public class TxConfig {

  @Bean
  public DataSource dataSource() throws PropertyVetoException {
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setUser("root");
    dataSource.setPassword("root");
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
    return dataSource;
  }

  @Bean
  public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
    return jdbcTemplate;
  }

  @Bean // 配置事务管理器
  public PlatformTransactionManager transactionManager() throws PropertyVetoException {
    return new DataSourceTransactionManager(dataSource());
  }
}
```

