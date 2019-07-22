# Spring Boot

## Spring Boot优点

- 更少的配置
- 使用java配置，避免使用XML配置
- 常用框架提供默认值
- 内置多种Web Server

## Spring Boot热部署

Spring Boot有一个开发工具`DevTools`模块

它有助于提高开发人员的生产力

Java开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器

开发人员可以重新加载Spring Boot上的更改，而无需重新启动服务器

## Spring Boot中的监视器

`actuator`是spring启动框架中的重要功能之一

`actuator`可帮助您访问生产环境中正在运行的应用程序的当前状态

`actuator`中已经内置了非常多的`Endpoints`（`health`、`info`、`beans`、`httptrace`、`shutdown`）等等，同时也允许我们自己扩展自己的端点

| id               | desc                                                         | Sensitive |
| :--------------- | ------------------------------------------------------------ | --------- |
| `auditevents`    | 显示当前应用程序的审计事件信息                               | Yes       |
| `beans`          | 显示应用Spring Beans的完整列表                               | Yes       |
| `caches`         | 显示可用缓存信息                                             | Yes       |
| `conditions`     | 显示自动装配类的状态及及应用信息                             | Yes       |
| `configprops`    | 显示所有 `@ConfigurationProperties` 列表                     | Yes       |
| `env`            | 显示 `ConfigurableEnvironment` 中的属性                      | Yes       |
| `flyway`         | 显示 Flyway 数据库迁移信息                                   | Yes       |
| `health`         | 显示应用的健康信息（未认证只显示`status`，认证显示全部信息详情） | No        |
| `info`           | 显示任意的应用信息（在资源文件写info.xxx即可）               | No        |
| `liquibase`      | 展示`Liquibase` 数据库迁移                                   | Yes       |
| `metrics`        | 展示当前应用的 `metrics` 信息                                | Yes       |
| `mappings`       | 显示所有 `@RequestMapping` 路径集列表                        | Yes       |
| `scheduledtasks` | 显示应用程序中的计划任务                                     | Yes       |
| `sessions`       | 允许从Spring会话支持的会话存储中检索和删除用户会话。         | Yes       |
| `shutdown`       | 允许应用以优雅的方式关闭（默认情况下不启用）                 | Yes       |
| `threaddump`     | 执行一个线程dump                                             | Yes       |
| `httptrace`      | 显示HTTP跟踪信息（默认显示最后100个HTTP请求 - 响应交换）     | Yes       |

## 如何在Spring Boot中禁用Actuator端点安全性

默认情况下，所有敏感的HTTP端点都是安全的，只有具有`ACTUATOR`角色的用户才能访问它们

使用`management.security.enabled = false`来禁用安全性

## 如何使用Spring Boot实现异常处理

通过实现一个`ControlerAdvice`类，来处理控制器类抛出的所有异常。

## Spring Boot常用注解

- @SpringBootApplication

  > 配置在启动类上
  >
  > 包含了`@ComponentScan`、`@Configuration`和`@EnableAutoConfiguration`注解

- @ComponentScan

  > 让Spring Boot扫描需要被SpringIoC容器管理的组件（`bean`）
  >
  > 让Spring Boot扫描`Configuration`类并把它加入到程序上下文

- @Configuration

  > 等同于spring的XML配置文件

- @EnableAutoConfiguration

  > 启用自动配置

- @RestController

  > 是`@Controller`和@`ResponseBody`的组合注解

- @ControllerAdvice

  > 包含`@Component`，统一处理异常

- @ExceptionHandler（Exception.class）

  > 用在方法上面表示遇到这个异常就执行以下方法

- @Conditional

  > 该注解可以根据是否满足某一个特定条件来决定是否创建某个特定的Bean
  >
  > 可以通过`@Conditional`注解来实现只有当某个Bean被已创建时（存在时）才会创建另外一个Bean
  >
  > 这样就可以依据这一特定的条件来控制Bean的创建行为，这样的话我们就可以利用这样一个特性来实现一些自动的配置

  `@Conditional`是Spring Boot实现`自动配置`的关键。在此基础上，springboot又创建了多个适用于不同场景的组合条件注解

  - `@ConditionalOnBean`：当上下文中有指定Bean的条件下进行实例化。
  - `@ConditionalOnMissingBean`：当上下文没有指定Bean的条件下进行实例化。
  - `@ConditionalOnClass`：当classpath类路径下有指定类的条件下进行实例化。
  - `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下进行实例化。
  - `@ConditionalOnWebApplication`：当项目本身是一个Web项目时进行实例化。
  - `@ConditionalOnNotWebApplication`：当项目本身不是一个Web项目时进行实例化。
  - `@ConditionalOnProperty`：当指定的属性有指定的值时进行实例化。
  - `@ConditionalOnExpression`：基于SpEL表达式的条件判断。
  - `@ConditionalOnJava`：当JVM版本为指定的版本范围时进行实例化。
  - `@ConditionalOnResource`：当类路径下有指定的资源时进行实例化。
  - `@ConditionalOnJndi`：在JNDI存在时触发实例化。
  - `@ConditionalOnSingleCandidate`：当指定的Bean在容器中只有一个，或者有多个但是指定了首选的Bean时触发实例化。

## Spring Boot为什么能自动配置

因为`@EnableAutoConfiguration`注解中导入了`AutoConfigurationImportSelector`类

该类实现了`ImportSelector`接口

当该类生效的时候会调用重写方法`selectImports()`

该方法中最终会调用`SpringFactoriesLoader`的`loadFactories()`方法

扫描所有依赖中的`META-INF/spring.factories`中的类

然后把这些`对应关系`全部放到一个`properties`对象中

然后加载`properties`对象中的类

比如项目中存在Redis依赖，项目启动的时候回去检查Redis依赖中的`META-INF/spring.factories`文件

```properties
org.springframework.data.repository.core.support.RepositoryFactorySupport=org.springframework.data.redis.repository.support.RedisRepositoryFactory
```

那么前面一段`RepositoryFactorySupport`就是Spring Boot中的`接口`或者`抽象类`

后面一段`RedisRepositoryFactory`就是`Redis`集成Spring Boot所提供的`实现类`

然后通过`RedisRepositoryFactory`这个核心类加载`Redis`的一些相关类

比如`RedisOperations`，然后这个类一旦初始化，就会使Spring Boot中的`autoconfigure`包中的`RedisAutoConfiguration`生效

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration
```

然后从配置文件中去读取Redis的相关配置，然后进一步的加载`Redis`

然后又像上面一样，一旦某个类被初始化，又会链式的引起其他配置

最终`Redis`配置完全了

项目中就直接可以使用`Redis`了

