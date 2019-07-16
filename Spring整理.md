

# Spring整理

## Spring IoC和DI的理解

- IoC

  > Inversion of Control
  >
  > 控制反转，将之前我们自己去new一个对象这件事，交给Spring来帮我们管理

- DI

  > Dependency Injection
  >
  > 依赖注入，在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件

## Spring中依赖注入的三种方式

- `构造器`注入
- `setter方法`注入
- `接口`注入

## Spring框架中的IoC

IoC容器由`beans`包和`context`包构成

`BeanFactory`是Spring IoC容器的核心接口，用来包装和管理前面提到的各种bean

`BeanFactory`接口提供了先进的配置机制，使得任何类型的对象的配置成为了可能

## BeanFactory和ApplicationContext有什么区别

- `BeanFactory`采取延迟加载，第一次getBean时才会初始化Bean
- `ApplicationContext`是会在加载配置文件时初始化Bean。

`ApplicationContext`是对`BeanFactory`扩展，它可以进行

- 国际化处理
- 事件传递
- bean自动装配
- 各种不同应用层的Context实现

开发中基本都在使用`ApplicationContext`, web项目使用`WebApplicationContext` ，**很少**用到`BeanFactory`

我们常用的`ClassPathXmlApplicationContext`就间接地实现了`ApplicationContext`接口

下面是三种常用`ApplicationContext` 实现方式

1. ClassPathXmlApplicationContext

   > 从`classpath`的XML配置文件中读取上下文

2. FileSystemXmlApplicationContext

   > 由`文件系统`中的XML配置文件读取上下文

3. XmlWebApplicationContext

   > 由`Web应用`的XML文件读取上下文

## Spring配置bean实例化有哪些方式

- 类构造器实例化（默认无参数）

  ```xml
  <bean id="bean1" class="cn.itcast.spring.b_instance.Bean1"></bean>
  ```

- 静态工厂方法实例化（简单工厂模式）

  ```xml
  //下面这段配置的含义：调用Bean2Factory的getBean2方法得到bean2
  <bean id="bean2" class="cn.itcast.spring.b_instance.Bean2Factory" factory-method="getBean2"></bean>
  ```

- 实例工厂方法实例化（工厂方法模式）

  ```xml
  //先创建工厂实例bean3Facory，再通过工厂实例创建目标bean实例
  <bean id="bean3Factory" class="cn.itcast.spring.b_instance.Bean3Factory"></bean>
  <bean id="bean3" factory-bean="bean3Factory" factory-method="getBean3"></bean>
  ```

## Spring有几种配置方式

1. 基于`XML`的配置

   > applicationContext.xml中配置<bean id="xxx">

2. 基于`注解`的配置

   > 使用`@Bean`、`@Configuration`等注解
   >
   > 开启注解配置`<context:annotation-config/>`

3. 基于`Java`的配置

   > 配合`注解`使用

## Spring Bean的生命周期

`BeanFactory` 负责管理在spring容器中被创建的bean的生命周期

Bean的生命周期由`2组回调（CallBack）方法`组成

- 初始化之后
- 销毁之前

Spring框架提供 `4` 种方法来管理Bean的生命周期事件

- `InitializingBean`和`DisposableBean`回调接口

- 针对特殊行为的其他`Aware`接口

- Bean配置文件中的`init-method`方法和`destroy-method`方法，注：`destroy-method`方法仅对`singleton`有效

  ```xml
  <beans>
  	<bean id="demoBean" class="com.howtodoinjava.task.DemoBean" init-method="customInit" destroy-method="customDestroy"></bean>
  </beans>
  ```

- `@PostConstruct`和@`PreDestroy`注解方式

## Bean的完整生命周期（了解可以更好理解Bean的机制）

1. instantiate bean对象实例化
2. populate properties 封装属性
3. 如果Bean实现`BeanNameAware` 执行`setBeanName()`
4. 如果Bean实现`BeanFactoryAware` 或者 `ApplicationContextAware` 设置工厂 `setBeanFactory()` 或者上下文对象 `setApplicationContext()`
5. 如果存在Bean实现 `BeanPostProcessor`，执行`postProcessBeforeInitialization()`。`BeanPostProcessor`接口提供钩子函数，用来动态扩展修改Bean
6. 如果Bean实现`InitializingBean` 执行 `afterPropertiesSet()`
7. 调用指定初始化方法 `init-method()`
8. 如果存在Bean实现 `BeanPostProcessor` ，执行`postProcessAfterInitialization()`
9. 执行业务处理
10. 如果Bean实现 `DisposableBean` 执行 `destroy()`
11. 调用指定销毁方法 `destroy-method()`

## Spring Bean的作用域

Spring容器中的bean可以分为 `5` 个范围

- `singleton`

  > 整个程序中只有一份

- `prototype`

  > 为每个bean的请求创建一份

- `request`

  > 为自客户端的网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收

- `session`

  > 每个session中有一个bean的实例，在session过期后，bean会随之失效

- `global-session`

  > 如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中

## Spring的核心类有哪些，各有什么作用

- `BeanFactory`：产生一个新的实例，可以实现单例模式
- `BeanWrapper`：提供统一的get及set方法
- `ApplicationContext`：提供框架的实现，包括BeanFactory的所有功能

## Spring里面applicationContext.xml文件能不能改成其他文件名

`ContextLoaderListener`是一个ServletContextListener

它在你的web应用启动的时候初始化

缺省情况下， 它会在`WEB-INF/applicationContext.xml`文件找Spring的配置

你可以通过定义一个元素名字为`contextConfigLocation`来改变Spring配置文件的位置

```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/abc.xml</param-value>
		</context-param>
	</listener-class>
</listener>
```

## Spring中单例Bean是线程安全的么

**不是**

Spring中没有对SIngleton的bean做任何多线程的封装处理

如果需要实现线程安全的话，需要开发者自行处理

实际上，大部分单例的bean一般都是`service`、`dao`之类的bean

这类bean并没有多种状态可变，所以就算多线程调用的也不会有什么安全问题

但是如果是某种状态较多的bean的话，则需要自行保证线程安全，比如将该bean的作用域设为`prototype`

## Spring如何处理线程并发问题

- Spring使用`ThreadLocal`解决线程安全问题

- 我们知道在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为`singleton`作用域

- 就是因为Spring对一些Bean（如`RequestContextHolder`、`TransactionSynchronizationManager`、`LocaleContextHolder`等）中非线程安全状态采用`ThreadLocal`进行处理，让它们也成为线程安全的状态，因为有状态的Bean就可以在多线程中共享了。

- `ThreadLocal`和`线程同步机制`都是为了解决多线程中相同变量的访问冲突问题。

- 在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。

- 而`ThreadLocal`则从另一个角度来解决多线程的并发访问。`ThreadLocal`会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。

- 因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。`ThreadLocal`提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进`ThreadLocal`。

- 由于`ThreadLocal`中可以持有任何类型的对象，低版本JDK所提供的`get()`返回的是Object对象，需要强制类型转换。但JDK5.0通过泛型很好的解决了这个问题，在一定程度地简化`ThreadLocal`的使用。

- 概括起来说

  > `同步机制`采用了	*以时间换空间*	的方式
  >
  > `ThreadLocal`采用了	*以空间换时间*	的方式
  >
  > 前者仅提供一份变量，让不同的线程排队访问，
  >
  > 而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

## Spring中注入一个Collection

- List

  ```xml
  <!-- java.util.List -->
  <property name="customList">
      <list>
          <value>INDIA</value>
          <value>Pakistan</value>
          <value>USA</value>
          <value>UK</value>
      </list>
  </property>
  ```

- Set

  ```xml
  <!-- java.util.Set -->
  <property name="customSet">
      <set>
          <value>INDIA</value>
          <value>Pakistan</value>
          <value>USA</value>
          <value>UK</value>
      </set>
  </property>
  ```

- map

  ```xml
  <!-- java.util.Map -->
  <property name="customMap">
      <map>
          <entry key="1" value="INDIA"/>
          <entry key="2" value="Pakistan"/>
          <entry key="3" value="USA"/>
          <entry key="4" value="UK"/>
      </map>
  </property>
  ```

- properties

  ```xml
  <!-- java.util.Properties -->
  <property name="customProperies">
      <props>
          <prop key="admin">admin@nospam.com</prop>
          <prop key="support">support@nospam.com</prop>
      </props>
  </property>
  ```

## Spring Bean的自动装配

- XML配置装配

  ```xml
  <bean id="employeeDAO" class="com.xxx.EmployeeDAOImpl" autowire="byName" />
  ```

- 注解装配

  > 使用`@Autowired`注解来自动装配指定的bean
  >
  > 前提是需要开启注解配置`<context:annotation-config />`
  >
  > 或者配置`AutowiredAnnotationBeanPostProcessor`，也能达到相同的效果

  ```xml
  <bean class ="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
  ```

## 5种自动装配模式

- no

  > 这是Spring框架的**默认设置**，在该设置下自动装配是关闭的，开发者需要自行在bean定义中用标签明确的设置依赖关系。

- byName

  > 该选项可以根据bean名称设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的名称自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- byType

  > 该选项可以根据bean类型设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的类型自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- constructor

  > 造器的自动装配和`byType模式`类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。

- autoDetect

  > 该模式自动探测使用构造器自动装配或者byType自动装配。首先，首先会尝试找合适的带参数的`构造器`，如果找到的话就是用构造器自动装配，如果在bean内部没有找到相应的构造器或者是无参构造器，容器就会自动选择`byTpe`的自动装配方式。

## @Required注解

`@Required`适用于属性的`setter`方法

表示某个属性必须赋值

如果没有赋值的话则会抛出`BeanInitializationException`异常

前提是开启了`注解配置`

或者配置了`RequiredAnnotationBeanPostProcesso`

## @Qualifier注解

配合`@Autowired`注解一起使用

当IoC容器中存在多个相同类型的Bean

此时需要使用`@Qualifie("beanName")`来指定具体装配那个类

## Spring的事物传播行为

Spring中提供了自己的事务管理机制

一般是使用`TransactionMananger`进行管理，可以通过Spring的注入来完成此功能

spring提供了几个关于事务处理的类：

- `TransactionDefinition` 

  > 事务属性定义

- `TranscationStatus` 

  > 代表了当前的事务，可以提交，回滚。

- `PlatformTransactionManager`

  > 这个是spring提供的用于管理事务的基础接口，其下有一个实现的抽象类 `AbstractPlatformTransactionManager`
  >
  > 我们使用的事务管理类例如 `DataSourceTransactionManager`等都是这个类的子类。

一般事务定义步骤：

```java
TransactionDefinition td =new TransactionDefinition();
TransactionStatus ts = transactionManager.getTransaction(td);
try{
	//do sth
	transactionManager.commit(ts);
}catch(Exception e){
	transactionManager.rollback(ts);
}
```

spring提供的事务管理可以分为两类

- 编程式

  > 比较灵活，但是代码量大，存在重复的代码比较多
  >
  > 编程式主要使用`TransactionTemplate`
  >
  > 省略了部分的提交，回滚，一系列的事务对象定义，需注入事务管理对象.

  ```java
  void add(){
  	transactionTemplate.execute(new TransactionCallback(){
          pulic Object doInTransaction(TransactionStatus ts){
      	//do sth
          }
  	}
  }
  ```

- 声明式的

  > 声明式的比编程式的更灵活
  >
  > 使用TransactionProxyFactoryBean
  >
  > 围绕Poxy的动态代理 能够自动的提交和回滚事务

  ```java
  PROPAGATION_REQUIRED–支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
  
  PROPAGATION_SUPPORTS–支持当前事务，如果当前没有事务，就以非事务方式执行。
  
  PROPAGATION_MANDATORY–支持当前事务，如果当前没有事务，就抛出异常。
  
  PROPAGATION_REQUIRES_NEW–新建事务，如果当前存在事务，把当前事务挂起。
  
  PROPAGATION_NOT_SUPPORTED–以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
  
  PROPAGATION_NEVER–以非事务方式执行，如果当前存在事务，则抛出异常。
  
  PROPAGATION_NESTED–如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与
  ```

## Spring AOP名词

- 切面（Aspect）

  > 一个关注点的模块化，这个关注点可能会横切多个对象
  >
  > 在Spring AOP中，切面可以使用通用类（基于模式的风格）或者在普通类中以 `@Aspect` 注解（@AspectJ风格）来实现

- 连接点（Joinpoint）

  > 在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候
  >
  >  在Spring AOP中，一个连接点总是代表一个方法的执行
  >
  >  通过声明一个`org.aspectj.lang.JoinPoint`类型的参数可以使通知`Advice`的主体部分获得连接点信息

- 通知（Advice）

  > 在切面的某个特定的连接点（Joinpoint）上执行的动作
  >
  > 通知有各种类型，其中包括`around`、`before`和`after`等通知
  >
  > 许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链

- 切入点（Pointcut）

  > 匹配连接点（Joinpoint）的规则
  >
  > 通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行
  >
  > `切入点表达式`如何和`连接点匹配`是AOP的核心：Spring缺省使用AspectJ切入点语法

- 引入（Introduction）

  > （也被称为内部类型声明（inter-type declaration））
  >
  > 声明额外的方法或者某个类型的字段
  >
  > Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 `IsModified` 接口，以便简化缓存机制

- 目标对象（Target Object）

  > 被一个或者多个切面（aspect）所通知（advise）的对象
  >
  > 也有人把它叫做被通知（advised）对象
  >
  > 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象

- AOP代理（AOP Proxy）

  > AOP框架创建的对象，用来实现切面契约（aspect contract）
  >
  > 在Spring中，AOP代理可以是`JDK动态代理`或者`CGLIB代理`
  >
  > 注意：Spring 2.0最新引入的基于模式（schema-based）风格和@AspectJ注解风格的切面声明，对于使用这些风格的用户来说，代理的创建是透明的

- 织入（Weaving）

  > 把切面（aspect）连接到其它的应用程序类型或者对象上，并创建一个被通知（advised）的对象
  >
  > 这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。 Spring和其他纯Java AOP框架一样，在运行时完成织入

## 通知（Advice）有哪些类型

- 前置通知（before）

  > 在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）

- 返回后通知（after-returning）

  > 在某连接点（join point）正常完成后执行的通知

- 抛出异常后通知（after-throwing）

  > 在方法抛出异常退出时执行的通知

- 后通知（after）

  > 当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）

-  环绕通知（around）

  > 包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。

环绕通知是最常用的一种通知类型

大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知

`切入点（pointcut）`和`连接点（join point）`匹配的概念是AOP的关键

这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次

例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）

## Spring框架事件机制

Spring的`ApplicationContext` 提供了支持事件和代码中监听器的功能。

我们可以创建bean用来监听在`ApplicationContext` 中发布的事件

`ApplicationEvent`类和在`ApplicationContext`接口中处理的事件

如果一个bean实现了`ApplicationListener`接口，当一个`ApplicationEvent` 被发布以后，bean会自动被通知

```java
//事件监听器
public class AllApplicationEventListener implements ApplicationListener < ApplicationEvent >{
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent)
    {
    	//process event
    }
}
```

```java
//事件本身
public class CustomApplicationEvent extends ApplicationEvent{
    public CustomApplicationEvent ( Object source, final String msg ){
        super(source);
        System.out.println("Created a Custom event");
    }
}
```

```java
//通过applicationContext接口的publishEvent()方法来发布自定义事件。
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext,"message");
applicationContext.publishEvent(customEvent);
```

## Spring 框架中都用到了哪些设计模式

- 工厂模式

  > Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象

- 代理模式

  > Spring AOP 功能的实现
  >
  > 比如`JdkDynamicAopProxy`和`Cglib2AopProxy`

- 单例模式

  > 在spring配置文件中定义的bean默认为单例模式。

- 模板方法模式

  > 用来解决代码重复的问题。比如RestTemplate, JmsTemplate, JpaTemplate。

- 装饰者模式

  > 一种类名中含有`Wrapper`，一种类名中含有`Decorator`，基本上都是动态地给一个对象添加一些额外的职责。 

- 观察者模式

  > Spring中Observer模式常用的地方是listener的实现。如`ApplicationListener`。

- 适配器模式

  > 在Spring的Aop中，使用的`Advice`（通知）来增强被代理类的功能。

- 策略模式

  > spring中在实例化对象的时候用到Strategy模式
  > 在`SimpleInstantiationStrategy`中有如下代码说明了策略模式的使用情况