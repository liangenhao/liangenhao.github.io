---
title: 03 Spring Boot 之自动配置
date: 2018-08-11
categories: Spring Boot
tags: [Spring Boot]
---

我们可以在`application.properties`/`application.yml`文件中或通过命令行指定各种的属性。可以配置的属性见官方参考文档：[Common application porperties](https://docs.spring.io/spring-boot/docs/1.5.14.RELEASE/reference/htmlsingle/#common-application-properties)。 

<!-- more -->

## 1 自动配置原理

### 1.1 @EnableAutoConfiguration

Spring Boot 启动的时候，加载主配置类，使用`@EnableAutoConfiguration`（`@SpringBootApplication`组合注解里的一个），开启了自动配置功能。

`@EnableAutoConfiguration`注解作用是利用`EnableAutoConfigurationImportSelector`选择器给容器中导入一些组件。该选择器扫描所有jar包类路径下的`META-INF/spring.factories`文件。获取文件中key为`EnableAutoConfiguration`类名对应的值，然后将他们添加到容器中：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```

每一个这样的`XxxAutoConfiguration`类都是容器中的一个组件，作用就是用来进行自动配置。

### 1.2 以 HttpEncodingAutoConfiguration 为例

```java
// 这是一个配置类
@Configuration
// 启用指定类的ConfigurationProperties，将配置文件和HttpEncodingProperties绑定，并把HttpEncodingProperties添加到容器中。
@EnableConfigurationProperties(HttpEncodingProperties.class)
// 判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication
// 判断当前项目有没有CharacterEncodingFilter类，如果有，当前配置类生效
// CharacterEncodingFilter：springmvc中处理乱码的过滤器
@ConditionalOnClass(CharacterEncodingFilter.class)
// 判断配置文件中是否存在spring.http.encoding.enabled这个配置。matchIfMissing表示如果不存在，判断也成立（即使配置文件中不配置spring.http.encoding.enabled，当前配置类也生效）
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {
    // HttpEncodingProperties和配置文件进行映射了，可以直接取里面的值。
    private final HttpEncodingProperties properties;
	// 当只有一个有参构造器的情况下，参数的值就会从容器中获取
    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
        this.properties = properties;
    }

    @Bean
    // 当容器中没有CharacterEncodingFilter类，配置生效
    @ConditionalOnMissingBean(CharacterEncodingFilter.class)
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        return filter;
    }
}
```

> 根据当前不同的条件判断，决定这个配置类是否生效。一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这个组件的属性是从对应的properties类中获取的，这些properties类里面的每一个属性又是和配置文件绑定的。

`@HttpEncodingProperties`：**所有可以在配置文件中配置的属性都是在`XxxProperties`类中封装。**因此，配置文件中可以配置的数据就可以参照每一个自动配置的属性类。

```java
// 从配置文件中获取指定的值，并和bean进行绑定
@ConfigurationProperties(prefix = "spring.http.encoding")
public class HttpEncodingProperties {}
```

### 1.3 总结

`XxxAutoConfiguration`：自动配置类，配置类一旦生效，就会给容器中添加组件。

`XxxProperties`：封装配置文件中相关的属性。

一、Spring Boot 启动时会加载大量的自动配置类。

二、自动配置类生效时已经自动配置了一些组件，已经配置的组件，就不需要我们自己配置了。

三、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性值，我们就可以在配置文件中指定这些属性值。



## 2 @Conditional 派生注解

Spring 框架有`@Conditional`条件注解，只有当条件注解中的条件生效，对应的配置类配置的内容才会生效。

Spring Boot 对条件注解进行了扩展：

| @Conditional扩展注解              | 作用（判断是否满足当前指定条件）                 |
| --------------------------------- | ------------------------------------------------ |
| `@ConditionalOnJava`              | 系统的java版本是否符合要求                       |
| `@ConditionalOnBean`              | 容器中存在指定Bean；                             |
| `@ConditionalOnMissingBean`       | 容器中不存在指定Bean；                           |
| `@ConditionalOnExpression`        | 满足SpEL表达式指定                               |
| `@ConditionalOnClass`             | 系统中有指定的类                                 |
| `@ConditionalOnMissingClass`      | 系统中没有指定的类                               |
| `@ConditionalOnSingleCandidate`   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| `@ConditionalOnProperty`          | 系统中指定的属性是否有指定的值                   |
| `@ConditionalOnResource`          | 类路径下是否存在指定资源文件                     |
| `@ConditionalOnWebApplication`    | 当前是web环境                                    |
| `@ConditionalOnNotWebApplication` | 当前不是web环境                                  |
| `@ConditionalOnJndi`              | JNDI存在指定项                                   |

## 3 自动配置报告

自动配置类只有在一定的条件下才能生效。我们怎么才能知道哪些自动配置类生效了，哪些没有生效呢？

此时我们可以开启Spring Boot 的 debug模式。

在`application.properties`/`application.yml`配置文件中配置：`debug=true`。这样可以在控制台打印自动配置报告。

 