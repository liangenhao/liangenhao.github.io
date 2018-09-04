---
title: 08 Spring Boot 之数据访问
date: 2018-09-05
categories: Spring Boot
tags: [Spring Boot]
---

## 1 整合 JDBC

使用JDBC需要导入：`spring-boot-starter-jdbc`和数据库驱动依赖。

一、【数据库连接配置】：

访问数据库，只需要在`application.properties`/`application.yml`中配置：

```yaml
spring:
    datasource:
    username: root
    password: root
    url: jdbc:mysql://127.0.0.1:3306/jdbc
    driver‐class‐name: com.mysql.jdbc.Driver
```

> 数据源相关配置参考：`DataSourceProperties`。

<!-- more -->

二、【自动配置原理】：

1、`DataSourceConfiguration`根据配置创建数据源，默认使用Hikari连接池。可以使用`spring.datasource.type`指定自定义的数据源类型。

> Spring Boot 1.5.x 的默认数据源是Tomcat数据源连接池。Spring Boot 2.x改成了Hikari。

> Spring Boot 默认支持`tomcat.jdbc.pool.DataSource`、`HikariDataSource`、`dbcp.BaseDataSource`和自定义数据源。

2、`DataSourceAutoConfiguration`中有一个`DataSourceInitializer`，是一个`ApplicationListener`。

作用：1）运行建表语句。2）、运行插入数据的语句。

> 默认文件名称规则：建表sql：`schema-*.sql`。数据sql：`data-*.sql`。
>
> 如果自定义sql文件名：可以使用`spring.datasource.schema`来指定文件（可以指定多个）。

3、自动配置`JdbcTemplate`用来操作数据库。

## 2 使用自定义数据源：druid

一、【引入druid数据源依赖】：

```xml
<dependency>
	<groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.8</version>
</dependency>
```

二、【指定数据源的类型】：

使用`spring.datasource.type`来制定数据源的类型。

```yaml
spring:
	datasource:
		type: com.alibaba.druid.pool.DruidDataSource
```

三、【druid配置】：

通过一、二两步的操作，数据源已经切换到druid了，但如果需要对druid连接池做一些配置：

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
	# druid配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
	# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

以上druid配置直接写在配置文件里是不生效的。需要通过配置类，将这个配置想和`DataSource`绑定。

```java
@Configuration
public class DruidConfig {
	@ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
    return new DruidDataSource();
    }
}
```

> 使用`@ConfigurationProperties(prefix = "spring.datasource")`将以`spring.datasource`为前缀的配置想和`DruidDataSource`绑定。

四、【配置druid的监控】：

1、配置一个管理后台的`Servlet`：

```java
@Bean
public ServletRegistrationBean statViewServlet(){
    ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
    Map<String,String> initParams = new HashMap<>();
    initParams.put("loginUsername","admin");
    initParams.put("loginPassword","123456");
    initParams.put("allow","");//默认就是允许所有访问
    initParams.put("deny","192.168.15.21");
    bean.setInitParameters(initParams);
    return bean;
}
```

2、配置一个监控的`filter`：

```java
@Bean
public FilterRegistrationBean webStatFilter(){
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new WebStatFilter());
    Map<String,String> initParams = new HashMap<>();
    initParams.put("exclusions","*.js,*.css,/druid/*");
    bean.setInitParameters(initParams);
    bean.setUrlPatterns(Arrays.asList("/*"));
    return bean;
}
```

## 3 整合 Mybatis

需要引入`mybatis-spring-boot-starter`依赖和数据库驱动依赖。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 3.1 注解版

一、【常用注解】：

`@Mapper`：标注该类是操作数据库的mapper类。

`@MapperScan`：Mapper类的包扫描。

`@Select`：查询。

`@Delete`：删除。

`@Insert`：插入。

`@Update`：更新。

`@Options`：能够设置缓存时间，能够为对象生成自增的主键值 

二、【自定义mybatis配置规则】：

给容器添加一个`ConfigurationCustomizer`。

```java
@org.springframework.context.annotation.Configuration
@MapperScan(value = "com.enhao.springboot.mapper")
public class MyBatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){
            @Override
            public void customize(Configuration configuration) {
                // 开启下划线转驼峰
            	configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

> 可以使用`@MapperScan`批量扫描所有的Mapper接口。就不用在每个mapper接口上加`@Mapper`注解。

【小技巧】：

1. 增加控制台打印sql语句：

   在`application.properties`/`application.yml`配置：

   ```properties
   mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
   ```



### 3.2 配置文件版

一、【指定配置文件位置】：

```yaml
mybatis:
	config-location: classpath: mybatis/mybatis-config.xml
	mapper-locations: classpath: mybatis/mapper/*.xml
```

> `mybatis.config-location`指定全局配置文件。
>
> `mybatis.mapper-locations`指定mapper映射文件的位置。

## 4 整合 JPA

需要引入`spring-boot-starter-data-jpa`和依赖和数据库驱动依赖。

一、【对JPA的配置】：

使用`spring.jpa`对JPA进行配置。

```yaml
spring:
	jpa:
		hibernate:
			ddl-auto: update # 数据表的生成策略
			show-sql: true
```

> 所有关于JPA的配置，在`JPAProperties`中。

二、【编写模型和DAO接口】：

模型采用JPA规范的注解进行标注。

编写DAO接口，继承 `JPARepository`接口，就有了增删改查和分页排序的功能，不需要实现类。

